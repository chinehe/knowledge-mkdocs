[基础篇005](StarRocks基础篇005_索引.md) 讲的排序键和各类索引，解决的是"怎么少扫数据"。但还有一类查询，靠索引也很难做快：按主键高并发地点查整行。列存要把一行的每一列分别从不同的列文件里读出来再拼起来，列越多越吃亏。这一篇讲 StarRocks 为这类场景准备的存储方式：行列混存。

> 本文内容整理自 StarRocks 官方文档：[行列混存](https://docs.starrocks.io/zh/docs/table_design/hybrid_table/)。

## 1. 列存与行列混存

StarRocks 作为 OLAP 数据库，原本用**列式存储**，能提升聚合查询这类复杂查询的性能。v3.2.3 起，StarRocks 还支持**行列混存**（Beta）：数据同时按行和按列存储。它适合基于主键的高并发、低延迟点查和部分列更新，同时保留了接近列存的分析能力。行列混存还支持 prepared statement，能提高查询性能和安全性。

| 对比项 | 列式存储 | 行列混存 |
|---|---|---|
| **存储方式** | 只按列存 | 按行和按列都存。表里多一个隐藏的二进制列 `__row`：写入时，把每行所有 value 列的值编码后写入 `__row`。因此有额外的存储开销 |
| **适合的场景** | 复杂数据分析，例如海量数据上的聚合分析、多表 JOIN；字段很多的宽表，但查询只涉及少数几列 | 同时支持行存和列存的场景。行存场景包括：基于主键的高并发点查；字段少的表上查询大部分字段；部分列更新（更新多个列、少量行） |

## 2. 基本用法

### 2.1. 建表

先开启 FE 配置项 `enable_experimental_rowstore`，再在建表时把 `PROPERTIES` 中的 `store_type` 设为 `column_with_row`。

```sql
ADMIN SET FRONTEND CONFIG ("enable_experimental_rowstore" = "true");

CREATE TABLE users (
    id      bigint NOT NULL,
    country string,
    city    string,
    revenue bigint
)
PRIMARY KEY (id)
DISTRIBUTED BY HASH (id)
PROPERTIES ("store_type" = "column_with_row");
```

建表要求：

* 表必须是主键表；
* `__row` 列的长度不能超过 1 MB；
* v3.2.4 起，支持 BITMAP、HLL、JSON、ARRAY、MAP、STRUCT 类型的列。

### 2.2. 增删改

和列存表一样，可以用导入和 DML 语句插入、删除、更新数据。

```sql
INSERT INTO users (id, country, city, revenue) VALUES
    (1, 'USA', 'New York', 5000),
    (2, 'UK', 'London', 4500),
    (3, 'France', 'Paris', 6000),
    (4, 'Germany', 'Berlin', 4000),
    (5, 'Japan', 'Tokyo', 7000),
    (6, 'Australia', 'Sydney', 7500);

DELETE FROM users WHERE id = 6;
UPDATE users SET revenue = 6500 WHERE id = 4;
```

### 2.3. 点查走短路径

点查会走"短路径"（short circuit），直接查行存数据，提高查询性能。

1. 确认开启了查询短路：

    ```sql
    SHOW VARIABLES LIKE '%enable_short_circuit%';
    -- 未开启时：
    SET enable_short_circuit = true;
    ```

2. 查询满足条件时才走短路径：`WHERE` 子句的条件列**包含全部主键列**，并且运算符是 `=` 或 `IN`。条件列除了全部主键列之外，还可以包含其他列。

    ```sql
    SELECT * FROM users WHERE id = 1;
    ```

3. 用 `EXPLAIN` 验证，查询计划中出现 `Short Circuit Scan: true` 说明走了短路径：

    ```sql
    EXPLAIN SELECT * FROM users WHERE id = 1;
    -- ...
    --   0:OlapScanNode
    --      TABLE: users
    --      PREDICATES: 1: id = 1
    --      partitions=1/1
    --      tabletRatio=1/6
    --      Short Circuit Scan: true
    ```

### 2.4. Prepared statement

```sql
PREPARE select_all_stmt   FROM 'SELECT * FROM users';
PREPARE select_by_id_stmt FROM 'SELECT * FROM users WHERE id = ?';

SET @id1 = 1, @id2 = 2;

EXECUTE select_by_id_stmt USING @id1;
EXECUTE select_by_id_stmt USING @id2;
```

## 3. 限制

* 存算分离集群暂不支持行列混存；
* v3.2.4 起，行列混存的表支持用 `ALTER TABLE` 修改；
* 查询短路目前只适合"定时批量导入、导入完成后再查询"的场景。数据写入的 apply 阶段可能和查询短路产生索引互斥，导致写入阻塞短路查询，影响写入期间点查的响应时间；
* 行列混存会明显增加存储占用：数据存了行、列两份，而且行存的压缩率可能不如列存；
* 导入的耗时和资源消耗会增加；
* 可以作为在线服务的一种方案，但性能可能比不上成熟的 OLTP 数据库；
* 不支持依赖列存的功能，比如列模式的部分列更新；
* 必须是主键表。

## 4. 小结

* 行列混存（v3.2.3 起，Beta）在列存之外多存一份整行编码的隐藏列 `__row`，只能用于主键表，建表时设置 `"store_type" = "column_with_row"`。
* `WHERE` 条件覆盖全部主键列、且用 `=` 或 `IN` 的点查会走短路径，直接读行存；`EXPLAIN` 中的 `Short Circuit Scan: true` 可以验证。
* 代价是存储占用和导入开销增加，而且写入期间可能影响点查延迟，更适合"批量导入、之后查询"的在线服务。

到这里，表设计的各个决策（表类型、数据分布、压缩、排序键与索引、行列混存）就都讲完了。表建好之后，下一步是把数据可靠地导进来，这是导入篇的内容。
