

## 1. JSON概述

> 参考官方文档：[Data Type JSON](https://redis.io/docs/latest/develop/data-types/json/)

RedisJSON 是 Redis 的官方扩展模块，为 Redis 提供了原生的 JSON 数据类型支持。与将 JSON 序列化为字符串存储的传统方式不同，RedisJSON 允许直接存储、查询和操作 JSON 文档。

> 传递给命令的JSON值深度可达128。如果你向命令传递嵌套级别大于128的JSON值，命令会返回错误。

**Redis JSON的主要特性如下：**

* 对JSON标准的全面支持。
* 提供了用于选择/更新文档中元素的JSONPath语法。
* 文档以二进制数据形式存储在树状结构中，允许快速访问子元素。无需传输整个对象即可获取嵌套值，特别适合大型 JSON 文档。
* 提供了所有JSON值类型的原子操作。


可以将 RedisJSON 想象成一个智能文件系统：

> - `$` 路径符号就像文件系统的根目录 `/`
> - `$.user.name` 路径如同访问 `/user/name` 文件
> - 修改单个字段就像编辑文件部分内容，无需重写整个文件

## 2. 安装Redis JSON

Redis JSON 是一个 Redis 模块，需要单独安装和加载。



这里演示如何以安装包方式安装Redis JSON：

1. 下载预编译包

   ```shell
   # 去安装目录
   cd /usr/src
   
   # 下载
   wget https://redismodules.s3.amazonaws.com/rejson/rejson.Linux-rhel7-x86_64.2.4.7.zip
   ```

2. 解压

   ```shell
   unzip rejson.Linux-rhel7-x86_64.2.4.7.zip -d ./rejson2.4.7/
   ```

3. 查看内容

   ```shell
   cd rejson2.4.7 # 进入目录
   ll # 查看目录文件
   
   # 输出
   -rw------- 1 root root    6951 4月  22 2023 module.json
   -rwxr-xr-x 1 root root 4425368 4月  22 2023 rejson.so
   ```

4. 将so文件放到目标文件夹

   ```shell
   # 我这里将它放到配置文件所在目录
   cp rejson.so /etc/redis/
   ```

   

5. 修改redis配置文件，配置Redis加载JSON模块

   ```shell
   loadmodule /etc/redis/rejson.so
   ```

6. 重启Redis服务

7. 验证，在redis客户端中执行module list命令

   ```shell
   127.0.0.1:6379> module list
   1) 1) "name"
      2) "ReJSON"
      3) "ver"
      4) (integer) 20407
      5) "path"
      6) "/etc/redis/rejson.so"
      7) "args"
      8) (empty array)
   ```


## 3. JSON 路径语法

JSON Path语法用于访问JSON中的特定元素。

由于没有JSON路径语法的标准，Redis基于常见的最佳实践，实现了自己的JSON路径语法。



Redis JSON支持两种查询语法：

* JSON Path语法：RedisJSON v2.0 引入
* 第一版遗留的路径语法：RedisJSON v1 引入

Redis JSON根据路径查询的语句的第一个字符来判定使用哪种语法。如果第一个字符为`$`，则使用JSONPath语法，否则默认使用遗留路径语法。



### 2.1.  JSONPath 语法

#### 2.1.1. 概述

Redis JSON v2.0引入了JSONPath 语法，遵循了Goessner在他文章中描述的语法。

> 文章链接：[JsonPath](https://goessner.net/articles/JsonPath/)

JSONPath查询可以间接到JSON文档中的多个位置。在这种情况下，JSON命令会将操作应用到所有可能的位置。这相比传统的路径查询是一个重大改进，后者只在第一个路径上执行操作。

新语法支持括号表示法，允许使用特殊字符串如`:`或者键中的空白字符。



如果要在CLI查询中包含双引号，要用单引号包住JSONPath。例如：

```shell
JSON.GET store '$.inventory["mountain_bikes"]'
```

---

#### 2.1.2. 语法表

**JSONPath 语法表：**

| 语法元素           | 描述                                                         |
| ------------------ | ------------------------------------------------------------ |
| `$`                | 最外层JSON元素的根结点。                                     |
| `.`、`[]`          | 选择一个子元素                                               |
| `..`               | 向下递归JSON文档                                             |
| `*`                | 返回所有元素                                                 |
| `[]`               | 下标操作符，访问数组元素                                     |
| `[,]`              | 选择多个元素                                                 |
| `[start:end:step]` | 数组切片<br/>start、end、step为索引值<br/>索引值可以忽略，忽略时使用默认值。它们的默认值分别为：第一个字符、最后一个字符、1<br/>`[*]`和`[:]`选择所有元素 |
| `?()`              | 过滤JSON对象或数组。<br/>支持比较操作符号：`==`, `!=`, `<`, `<=`, `>`, `>=`, `=~`<br/>支持位运算符：`&&`, `||`<br/>支持括号：(`, `) |
| `()`               | 脚本表达式                                                   |
| `@`                | 当前元素，在过滤器和脚本表达式中使用                         |

#### 2.1.3. 示例

##### 2.1.3.1. 创建示例JSON

假设我们要操作的JSON文档存储了商店库存中商品的详细信息，如下：

```json
{
    "inventory": {
        "mountain_bikes": [
            {
                "id": "bike:1",
                "model": "Phoebe",
                "description": "This is a mid-travel trail slayer that is a fantastic daily driver or one bike quiver. The Shimano Claris 8-speed groupset gives plenty of gear range to tackle hills and there\u2019s room for mudguards and a rack too.  This is the bike for the rider who wants trail manners with low fuss ownership.",
                "price": 1920,
                "specs": {"material": "carbon", "weight": 13.1},
                "colors": ["black", "silver"],
            },
            {
                "id": "bike:2",
                "model": "Quaoar",
                "description": "Redesigned for the 2020 model year, this bike impressed our testers and is the best all-around trail bike we've ever tested. The Shimano gear system effectively does away with an external cassette, so is super low maintenance in terms of wear and tear. All in all it's an impressive package for the price, making it very competitive.",
                "price": 2072,
                "specs": {"material": "aluminium", "weight": 7.9},
                "colors": ["black", "white"],
            },
            {
                "id": "bike:3",
                "model": "Weywot",
                "description": "This bike gives kids aged six years and older a durable and uberlight mountain bike for their first experience on tracks and easy cruising through forests and fields. A set of powerful Shimano hydraulic disc brakes provide ample stopping ability. If you're after a budget option, this is one of the best bikes you could get.",
                "price": 3264,
                "specs": {"material": "alloy", "weight": 13.8},
            },
        ],
        "commuter_bikes": [
            {
                "id": "bike:4",
                "model": "Salacia",
                "description": "This bike is a great option for anyone who just wants a bike to get about on With a slick-shifting Claris gears from Shimano\u2019s, this is a bike which doesn\u2019t break the bank and delivers craved performance.  It\u2019s for the rider who wants both efficiency and capability.",
                "price": 1475,
                "specs": {"material": "aluminium", "weight": 16.6},
                "colors": ["black", "silver"],
            },
            {
                "id": "bike:5",
                "model": "Mimas",
                "description": "A real joy to ride, this bike got very high scores in last years Bike of the year report. The carefully crafted 50-34 tooth chainset and 11-32 tooth cassette give an easy-on-the-legs bottom gear for climbing, and the high-quality Vittoria Zaffiro tires give balance and grip.It includes a low-step frame , our memory foam seat, bump-resistant shocks and conveniently placed thumb throttle. Put it all together and you get a bike that helps redefine what can be done for this price.",
                "price": 3941,
                "specs": {"material": "alloy", "weight": 11.6},
            },
        ],
    }
}
```

在Redis CLI中使用如下的命令来创建它：

```shell
JSON.SET bikes:inventory $ '{ "inventory": { "mountain_bikes": [ { "id": "bike:1", "model": "Phoebe", "description": "This is a mid-travel trail slayer that is a fantastic daily driver or one bike quiver. The Shimano Claris 8-speed groupset gives plenty of gear range to tackle hills and there\'s room for mudguards and a rack too. This is the bike for the rider who wants trail manners with low fuss ownership.", "price": 1920, "specs": {"material": "carbon", "weight": 13.1}, "colors": ["black", "silver"] }, { "id": "bike:2", "model": "Quaoar", "description": "Redesigned for the 2020 model year, this bike impressed our testers and is the best all-around trail bike we\'ve ever tested. The Shimano gear system effectively does away with an external cassette, so is super low maintenance in terms of wear and tear. All in all it\'s an impressive package for the price, making it very competitive.", "price": 2072, "specs": {"material": "aluminium", "weight": 7.9}, "colors": ["black", "white"] }, { "id": "bike:3", "model": "Weywot", "description": "This bike gives kids aged six years and older a durable and uberlight mountain bike for their first experience on tracks and easy cruising through forests and fields. A set of powerful Shimano hydraulic disc brakes provide ample stopping ability. If you\'re after a budget option, this is one of the best bikes you could get.", "price": 3264, "specs": {"material": "alloy", "weight": 13.8} } ], "commuter_bikes": [ { "id": "bike:4", "model": "Salacia", "description": "This bike is a great option for anyone who just wants a bike to get about on With a slick-shifting Claris gears from Shimano\'s, this is a bike which doesn\'t break the bank and delivers craved performance. It\'s for the rider who wants both efficiency and capability.", "price": 1475, "specs": {"material": "aluminium", "weight": 16.6}, "colors": ["black", "silver"] }, { "id": "bike:5", "model": "Mimas", "description": "A real joy to ride, this bike got very high scores in last years Bike of the year report. The carefully crafted 50-34 tooth chainset and 11-32 tooth cassette give an easy-on-the-legs bottom gear for climbing, and the high-quality Vittoria Zaffiro tires give balance and grip.It includes a low-step frame , our memory foam seat, bump-resistant shocks and conveniently placed thumb throttle. Put it all together and you get a bike that helps redefine what can be done for this price.", "price": 3941, "specs": {"material": "alloy", "weight": 11.6} } ] }}'
```

##### 2.1.3.2. 访问示例



**可以使用`*`操作符获取inventory的所有元素：**

```shell
JSON.GET bikes:inventory $.inventory.*
"[[{\"id\":\"bike:1\",\"model\":\"Phoebe\",\"description\":\"This is a mid-travel trail slayer...
```

---



## 2. 相关命令

### 2.1 基本操作

#### 2.1.1 JSON.SET：创建/更新文档

**语法**：
```
JSON.SET <key> <path> <json> [NX | XX]
```

**参数说明**：
- `key`：Redis 键名
- `path`：JSONPath 路径（`$` 表示根节点）
- `json`：有效的 JSON 值
- `NX`：仅当路径不存在时设置
- `XX`：仅当路径存在时设置

**使用示例**：
```bash
## 创建完整JSON文档
JSON.SET user $ '{"name":"John", "age":30, "city":"New York"}'

## 更新嵌套字段
JSON.SET user $.age 31

## 条件更新（仅当字段存在）
JSON.SET user $.preferences.theme "dark" XX
```

> 💡 **初学者小贴士**
> 当看到 `$.user.name` 这样的路径时，可以把 `$` 想象成 JSON 文档的“起点站”，
> 后续的 `.user.name` 就是沿着“地铁线路”逐站查找，最终到达目标数据节点。

#### 2.1.2 JSON.GET：查询文档

**语法**：
```
JSON.GET <key> [path]
```

**参数说明**：
- `path`：可选路径，省略时返回整个文档

**使用示例**：
```bash
## 获取完整文档
JSON.GET user

## 获取特定字段
JSON.GET user $.name

## 获取多个字段（数组形式）
JSON.GET user $.name $.age
```

### 2.2 数值操作

#### 2.2.1 JSON.NUMINCRBY：数值递增

**语法**：
```
JSON.NUMINCRBY <key> <path> <number>
```

**使用示例**：
```bash
## 用户积分+10
JSON.NUMINCRBY user $.points 10

## 购物车商品数量+1
JSON.NUMINCRBY cart $.items[0].quantity 1
```

### 2.3 字符串操作

#### 2.3.1 JSON.STRAPPEND：字符串追加

**语法**：
```
JSON.STRAPPEND <key> <path> <string>
```

**使用示例**：
```bash
## 给用户描述追加内容
JSON.STRAPPEND user $.bio " - Redis enthusiast"
```

### 2.4 数组操作

#### 2.4.1 JSON.ARRAPPEND：数组追加

**语法**：
```
JSON.ARRAPPEND <key> <path> <json> [json ...]
```

**使用示例**：
```bash
## 添加新订单
JSON.ARRAPPEND orders $.history '{"id":101, "amount":99.9}'

## 一次追加多个元素
JSON.ARRAPPEND user $.hobbies "hiking" "photography"
```

### 2.5 对象操作

#### 2.5.1 JSON.OBJKEYS：获取对象键列表

**语法**：
```
JSON.OBJKEYS <key> <path>
```

**使用示例**：
```bash
## 获取用户对象的所有字段名
JSON.OBJKEYS user $
## 返回 ["name", "age", "city"]
```

## 3. 内部实现原理

RedisJSON 采用 **二进制树结构** 存储 JSON 文档，其核心设计特点：

1. **内存优化**：
   - 共享相同前缀的路径节点
   - 原始数据类型直接存储（避免字符串转换开销）

2. **快速访问**：
   ```mermaid
   graph TD
     A[JSON Root] --> B["$.user"]
     A --> C["$.settings"]
     B --> D["$.user.name"]
     B --> E["$.user.age"]
     C --> F["$.settings.theme"]
   ```
   通过树形结构实现 O(1) 复杂度的路径查找

3. **原子更新机制**：
   - 修改操作直接在树节点上进行
   - 避免全量文档解析和序列化

> 💡 **初学者小贴士**
> 想象 JSON 文档是一棵圣诞树：
> - 树干是根节点 `$`
> - 树枝是各级路径（如 `.user`）
> - 装饰品是最终数据（如 `.name`）
> 修改某个装饰品时，只需调整对应树枝，无需重装整棵树