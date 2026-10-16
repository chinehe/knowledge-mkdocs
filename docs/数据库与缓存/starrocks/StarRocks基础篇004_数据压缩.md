[基础篇003](StarRocks基础篇003_数据分布.md) 决定了数据切成哪些 Tablet、放到哪些节点。数据真正落盘时，每一列都会先编码、再压缩。压缩算法选得合不合适，影响的是存储空间，以及导入和查询时的 I/O 与 CPU 开销。这一篇讲 StarRocks 支持哪些压缩算法、怎么选、怎么设置。

> 本文内容整理自 StarRocks 官方文档：[数据压缩](https://docs.starrocks.io/zh/docs/table_design/data_compression/)。

## 1. 压缩的作用与代价

StarRocks 支持对**表和索引数据**进行压缩：

* **好处**：节省存储空间；同时读写的数据量变小，能提升 I/O 密集型任务的性能；
* **代价**：压缩和解压缩需要额外的 CPU 资源。

## 2. 选择压缩算法

StarRocks 支持四种压缩算法，它们在压缩率和压缩/解压缩性能上各有取舍：

| 算法 | 压缩率排名 | 特点 |
|---|---|---|
| zlib | 1（最高） | 压缩比高，但数据高度压缩后，导入和查询性能会受一定影响 |
| Zstandard（zstd） | 2 | 压缩比与解压缩性能较均衡 |
| LZ4 | 3 | 压缩比与解压缩性能较均衡，**默认算法** |
| Snappy | 4（最低） | — |

选择建议：

* 如果对存储空间没有特殊要求，官方建议使用 **LZ4 或 Zstandard**；
* 存储空间特别紧张、能接受导入和查询性能有所下降时，再考虑 zlib；
* 数据类型也会影响压缩率，同一算法在不同类型的列上效果不同。

## 3. 设置压缩算法

压缩算法**只能在建表时设置，建表后无法修改**。建表时通过 `PROPERTIES` 的 `compression` 属性指定，不指定则默认使用 LZ4：

```sql
CREATE TABLE data_compression (
    id   INT       NOT NULL,
    name CHAR(200) NULL
)
UNIQUE KEY(id)
DISTRIBUTED BY HASH(id)
PROPERTIES ("compression" = "ZSTD");
```

建表后可以用 `SHOW CREATE TABLE` 查看表实际使用的压缩算法。

## 4. 小结

* 压缩节省存储、减少 I/O，代价是 CPU。
* 压缩率 zlib > Zstandard > LZ4 > Snappy；默认 LZ4，无特殊需求时选 LZ4 或 Zstandard。
* 压缩算法只能在建表时通过 `"compression"` 设置，建表后不能修改。

数据怎么存下来已经确定了，接下来看查询怎么在这些数据里快速定位到需要的行：[基础篇005 · 索引](StarRocks基础篇005_索引.md)。
