Redis Hashes（哈希）是一种**键值对集合**数据类型，可以使用哈希来表示基本对象、计数器分组等。

本文详细介绍了哈希数据类型的基础知识、核心特性；对哈希类型相关的所有命令都提供了详细的说明和示例；另外，文章还介绍了Redis 7.4引入的**哈希字段有效期**。

参考官方文档：[Redis Hashes](https://redis.io/docs/latest/develop/data-types/hashes/)

## 1. 哈希类型

Redis Hashes（哈希）是一种**键值对集合**数据类型，可以使用哈希来表示基本对象、计数器分组等。

你可以将多个字段-值对存储在一个Redis哈希键中。这类似于一个小型的键值存储系统，但所有内容都位于一个顶级键下。

> 可以将Redis Hashes想象成一本字典或电话簿。整个字典有一个名字（Redis键），里面包含很多词条（字段），每个词条都有其解释（值）。比如，你可以用一个名为"user:1000"的哈希来存储用户信息，其中包含"name"、"email"、"age"等字段。



**主要特性：**

* **结构：**键值对集合
* **性能：** 大部分Redis哈希命令为O(1)；小部分命令为O(n)，比如 [`HKEYS`](https://redis.io/docs/latest/commands/hkeys/)、[`HVALS`](https://redis.io/docs/latest/commands/hvals/)、[`HGETALL`](https://redis.io/docs/latest/commands/hgetall/) 以及大多数与到期相关的命令
* **限制：** 每个哈希最多可存储 4,294,967,295（2^32 - 1）个字段值对。
* **底层实现：**在字段数量较少且元素较小时使用ziplist（压缩列表），否则使用hashtable（哈希表）

> Redis 7.0版本后，Hashes的编码优化进一步增强，能够更智能地在不同编码之间转换，以达到最佳的内存和性能平衡。

## 2. 相关命令

哈希数据类型的相关命令，参考官方文档：[Redis Command Hashes](https://redis.io/docs/latest/commands/?group=hash)

### 2.1. 命令列表

按照基本的功能分类，哈希相关的全部命令如下；

* 设置键值对
  * `HSET`：创建或修改哈希中字段的值。
  * `HSETEX`：创建或修改哈希中一个或多个字段的值，并设置有效期（可选）。
  * `HSETNX`：当哈希中指定字段不存在时，设置字段的值。
  * `HINCRBY`：将哈希中指定字段的整数值加N。
  * `HINCRBYFLOAT`：将哈希中指定字段的浮点值加F。
  * `HDEL`：从哈希中删除一个或多个键值对。
  * `HMSET`：设置哈希中的多个字段值（弃用）。
* 获取键值对
  * `HGET`：获取哈希中指定字段的值。
  * `HGETDEL`： 获取并删除哈希中指定的键值对。
  * `HGETEX`：获取哈希中一个或多个键，并设置其有效期（可选）。
  * `HGETALL`：获取哈希中所有的键值对。
  * `HMGET`：获取哈希中所有字段的值
  * `HEXISTS`：检查哈希中是否存在指定字段。
  * `HSCAN`：对哈希值的键值对进行迭代。
  * `HKEYS`：获取哈希中的所有字段。
  * `HVALS`：返回哈希中所有值。
  * `HRANDFIELD`：返回哈希中一个或多个随机字段。
  * `HLEN`：获取哈希中的字段数量。
  * `HSTRLEN`：获取哈希中指定字段值的长度。
* 设置有效期
  * `HEXPIRE`：使用相对到期时间设置哈希字段的到期时间。
  * `HPEXPIRE`：使用该相对过期时间（毫秒）设置哈希字段的过期时间。
  * `HEXPIREAT`：使用绝对Unix时间戳（秒）设置哈希字段的到期时间。
  * `HPEXPIREAT`：使用该Unix时间戳（毫秒）设置哈希字段的过期时间。
  * `HPERSIST`：移除哈希中指定字段的有效期。
* 获取有效期
  * `HEXPIRETIME`：以Unix时间戳（秒）返回哈希字段的过期时间。
  * `HPEXPIRETIME`： 以Unix时间戳（毫秒）返回哈希字段的过期时间。
  * `HPTTL`：返回哈希字段的TTL（毫秒）。
  * `HTTL`：返回哈希字段的TTL（秒）。

### 2.2. 设置键值对

设置键值对相关命令，用于创建、修改、删除哈希字段值。

#### 2.2.1. HSET

语法格式：

```shell
HSET key field value [field value ...]
```

为哈希中指定的字段设置值。

* 如果哈希中不存在该字段，将自动创建。
* 如果哈希中存在该字段，将覆盖值。



示例：

```shell
> HSET myhash field1 "Hello"
(integer) 1
> HGET myhash field1
"Hello"
> HSET myhash field2 "Hi" field3 "World"
(integer) 2
> HGET myhash field2
"Hi"
> HGET myhash field3
"World"
> HGETALL myhash
1) "field1"
2) "Hello"
3) "field2"
4) "Hi"
5) "field3"
6) "World"
```



#### 2.2.2. HSETEX

语法格式：

```shell
HSETEX key 
	[FNX | FXX] 
	[EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | KEEPTTL]
  FIELDS numfields field value [field value ...]
```

为哈希中指定的字段设置值，并且设置有效时间（可选）。

* 如果哈希中不存在该字段，将自动创建。
* 如果哈希中存在该字段，将覆盖值。



可选参数：

* 执行条件
  * FNX：所有字段都不存在时才执行。
  * FXX：所有字段都存在时才执行。
* 有效期
  * EX：将有效期设置成指定的秒数。
  * PX：将有效期设置成指定的毫秒数。
  * EXAT：设置到期时间为指定的Unix时间（秒）。
  * PXAT：设置到期时间为指定的Unix时间（毫秒）。
  * KEEPTTL：保持原有的字段有效期不变。



示例：

```shell
redis> HSETEX mykey EXAT 1740470400 FIELDS 2 field1 "Hello" field2 "World"
(integer) 1
redis> HTTL mykey FIELDS 2 field1 field2
1) (integer) 55627
2) (integer) 55627
redis> HSETEX mykey FNX EX 60 FIELDS 2 field1 "Hello" field2 "World"
(integer) 0
redis> HSETEX mykey FXX EX 60 KEEPTTL FIELDS 2 field1 "hello" field2 "world"
(error) ERR Only one of EX, PX, EXAT, PXAT or KEEPTTL arguments can be specified
redis> HSETEX mykey FXX KEEPTTL FIELDS 2 field1 "hello" field2 "world"
(integer) 1
redis> HTTL mykey FIELDS 2 field1 field2
1) (integer) 55481
2) (integer) 55481
```



#### 2.2.3. HSETNX

语法格式：

```shell
HSETNX key field value
```

当哈希中指定的字段不存在时，为字段设置值。

* 如果哈希中不存在该字段，将自动创建。
* 如果哈希中存在该字段，则不执行操作。



示例：

```shell
redis> HSETNX myhash field "Hello"
(integer) 1
redis> HSETNX myhash field "World"
(integer) 0
redis> HGET myhash field
"Hello"
redis>
```



#### 2.2.4. HINCRBY

语法格式：

```shell
HINCRBY key field increment
```

将哈希中指定字段的整数值增加increment。

* 如果字段不存在，当成0处理，结果为increment。
* 如果字段存在，
  * 如果值为数字，结果为原值+increment
  * 如果值不是数字，报错。

示例：

```shell
redis> HSET myhash field 5
(integer) 1
redis> HINCRBY myhash field 1
(integer) 6
redis> HINCRBY myhash field -1
(integer) 5
redis> HINCRBY myhash field -10
(integer) -5
redis> 
```



#### 2.2.5. HINCRBYFLOAT

语法格式：

```shell
HINCRBYFLOAT key field increment
```

将哈希中指定字段的浮点数值增加increment。

* 如果字段不存在，当成0处理，结果为increment。
* 如果字段存在，
  * 如果值为双精度浮点数，结果为原值+increment
  * 如果值不是双精度浮点数，报错。



示例：

```shell
redis> HSET mykey field 10.50
(integer) 1
redis> HINCRBYFLOAT mykey field 0.1
"10.6"
redis> HINCRBYFLOAT mykey field -5
"5.6"
redis> HSET mykey field 5.0e3
(integer) 0
redis> HINCRBYFLOAT mykey field 2.0e2
"5200"
redis> 
```



#### 2.2.6 HDEL

语法格式：

```shell
HDEL key field [field ...]
```

从哈希中删除指定的一个或多个字段



示例：

```shell
HSET myhash field1 "foo"
(integer) 1
HDEL myhash field1
(integer) 1
HDEL myhash field2
(integer) 0
```



### 2.3. 获取键值对

获取键值对命令用于获取哈希中的键/值/键值对及其信息。

#### 2.3.1. HGET

语法格式：

```shell
HGET key field
```

获取哈希中指定字段的值。



示例：

```shell
> HSET myhash field1 "foo"
(integer) 1
> HGET myhash field1
"foo"
> HGET myhash field2
(nil)
```



#### 2.3.2. HGETDEL

语法格式：

```shell
HGETDEL key FIELDS numfields field [field ...]
```

获取并删除哈希中指定的一个或多个字段。



示例：

```shell
redis> HSET mykey field1 "Hello" field2 "World" field3 "!"
(integer) 3
redis> HGETALL mykey
1) "field1"
2) "Hello"
3) "field2"
4) "World"
5) "field3"
6) "!"
redis> HGETDEL mykey FIELDS 2 field3 field4
1) "!"
2) (nil)
redis> HGETALL mykey
1) "field1"
2) "Hello"
3) "field2"
4) "World"
redis> HGETDEL mykey FIELDS 2 field1 field2
1) "Hello"
2) "World"
redis> KEYS *
(empty array)
```



#### 2.3.3. HGETEX

语法格式：

```shell
HGETEX key 
	[EX seconds | PX milliseconds | EXAT unix-time-seconds | PXAT unix-time-milliseconds | PERSIST] 	FIELDS numfields field
  [field ...]
```

获取哈希中的一个或多个字段的值，并设置有效期（可选）。



可选参数：

* 有效期

  * EX：将有效期设置成指定的秒数。

  * PX：将有效期设置成指定的毫秒数。

  * EXAT：设置到期时间为指定的Unix时间（秒）。

  * PXAT：设置到期时间为指定的Unix时间（毫秒）。

  * PERSIST：移除与字段关联的有效期。



示例：

```shell
redis> HSET mykey field1 "Hello" field2 "World"
(integer) 2
redis> HGETEX mykey EX 120 FIELDS 1 field1
1) "Hello"
redis> HGETEX mykey EX 100 FIELDS 1 field2
1) "World"
redis> HTTL mykey FIELDS 2 field1 field2
1) (integer) 91
2) (integer) 85
redis> HTTL mykey FIELDS 3 field1 field2 field3 
1) (integer) 75
2) (integer) 68
3) (integer) -2
...
redis> HTTL mykey FIELDS 3 field1 field2 
1) (integer) -2
2) (integer) -2
redis> HGETALL myk
```



#### 2.3.4. HGETALL

语法格式：

```shell
HGETALL key
```

返回哈希中的所有字段和值。



示例：

```shell
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HGETALL myhash
1) "field1"
2) "Hello"
3) "field2"
4) "World"
```



#### 2.3.5. HMGET

语法格式：

```shell
HMGET key field [field ...]
```

获取哈希中指定的一个或多个字段的值。



示例：

```shell
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HMGET myhash field1 field2 nofield
1) "Hello"
2) "World"
3) (nil)
redis>
```



#### 2.3.6. HEXISTS

语法格式：

```shell
HEXISTS key field
```

检查哈希中是否存在指定字段。



示例：

```shell
redis> HSET myhash field1 "foo"
(integer) 1
redis> HEXISTS myhash field1
(integer) 1
redis> HEXISTS myhash field2
(integer) 0
redis> 
```



#### 2.3.7. HSCAN

语法格式：

```shell
HSCAN key cursor [MATCH pattern] [COUNT count] [NOVALUES]
```

对哈希值的键值对进行迭代。



参考官方文档：[Redis SCAN](https://redis.io/docs/latest/commands/scan/)



#### 2.3.8. HKEYS

语法格式：

```shell
HKEYS key
```

获取哈希中所有的字段名称。



示例：

```shell
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HKEYS myhash
1) "field1"
2) "field2"
redis>
```



#### 2.3.9. HVALS

语法格式：

```shell
HVALS key
```

获取哈希中所有的字段值。



示例：

```shell
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HVALS myhash
1) "Hello"
2) "World"
```



#### 2.3.10. HRANDFIELD

语法格式：

```shell
HRANDFIELD key [count [WITHVALUES]]
```

返回哈希中的一个随机字段名称。



可选参数：

* count 计数：
  * 如果大于0：返回随机字段数组（不重复），数组长度等于count和哈希字段数的较小值。
  * 如果小于0：返回随机字段数组（可以重复），数组长度等于count的绝对值。
* WITHVALUES：指定需要同时返回对应字段的值。



示例：

```shell
redis> HSET coin heads obverse tails reverse edge null
(integer) 3
redis> HRANDFIELD coin
"edge"
redis> HRANDFIELD coin
"heads"
redis> HRANDFIELD coin -5 WITHVALUES
1) "tails"
2) "reverse"
3) "heads"
4) "obverse"
5) "tails"
6) "reverse"
7) "edge"
8) "null"
9) "tails"
10) "reverse"
redis> 
```



#### 2.3.11. HLEN

语法格式：

```shell
HLEN key
```

获取哈希中的字段数量。



示例：

```shell
redis> HSET myhash field1 "Hello"
(integer) 1
redis> HSET myhash field2 "World"
(integer) 1
redis> HLEN myhash
(integer) 2
redis>
```



### 2.4. 字段有效期

#### 2.4.1. 关于字段有效期

Redis 7.4 引入了为哈希中单个字段设置到期时间的功能：

* 设置有效期
  * `HEXPIRE`：使用相对到期时间设置哈希字段的到期时间。
  * `HPEXPIRE`：使用该相对过期时间（毫秒）设置哈希字段的过期时间。
  * `HEXPIREAT`：使用绝对Unix时间戳（秒）设置哈希字段的到期时间。
  * `HPEXPIREAT`：使用该Unix时间戳（毫秒）设置哈希字段的过期时间。
  * `HPERSIST`：移除哈希中指定字段的有效期。
* 获取有效期
  * `HEXPIRETIME`：以Unix时间戳（秒）返回哈希字段的过期时间。
  * `HPEXPIRETIME`： 以Unix时间戳（毫秒）返回哈希字段的过期时间。
  * `HPTTL`：返回哈希字段的TTL（毫秒）。
  * `HTTL`：返回哈希字段的TTL（秒）。



Redis 8.0 引入了以下额外命令：

* HGETEX
* HSETEX

#### 2.4.2. 设置字段有效期

##### 2.4.2.1. HEXPIRE

语法格式：

```shell
HEXPIRE key seconds 
	[NX | XX | GT | LT] 
FIELDS numfields field
  [field ...]
```

设置哈希中一个或多个字段的有效期。



可选参数：

* 执行条件
  * NX：对于每个字段，仅在字段无到期时间时设置。
  * XX：对于每个字段，仅在字段已有有效期时设置。
  * GT：对于每个字段，仅在新有效期大于当前有效期时设置。
  * LT：对于每个字段，仅在新有效期小于当前有效期时设置。



示例：

```shell
HEXPIRE no-key 20 NX FIELDS 2 field1 field2
(nil)
HSET mykey field1 "hello" field2 "world"
(integer) 2
HEXPIRE mykey 10 FIELDS 3 field1 field2 field3
1) (integer) 1
2) (integer) 1
3) (integer) -2
HGETALL mykey
```



##### 2.4.2.2. HEXPIREAT

语法格式：

```shell
EXPIREAT key unix-time-seconds 
	[NX | XX | GT | LT] 
FIELDS numfields field [field ...]
```

使用绝对Unix时间戳（秒）设置哈希中一个或多个字段的到期时间。

> 可选参数的含义同HEXPIRE



示例：

```shell
redis> HSET mykey field1 "hello" field2 "world"
(integer) 2
redis> HEXPIREAT mykey 1715704971 FIELDS 2 field1 field2
1) (integer) 1
2) (integer) 1
redis> HTTL mykey FIELDS 2 field1 field2
1) (integer) 567
2) (integer) 567
```



##### 2.4.2.3. HPEXPIRE

语法格式：

```shell
HPEXPIRE key milliseconds 
	[NX | XX | GT | LT] 
FIELDS numfields field [field ...]
```

设置哈希中一个或多个字段的有效期（毫秒）。

> 可选参数的含义同HEXPIRE



示例：

```shell
redis> HSET mykey field1 "hello" field2 "world"
(integer) 2
redis> HPEXPIRE mykey 2000 FIELDS 2 field1 field2
1) (integer) 1
2) (integer) 1
redis> HGETALL mykey
(empty array)
```



##### 2.4.2.4. HPEXPIREAT

语法格式：

```shell
HPEXPIREAT key unix-time-milliseconds 
	[NX | XX | GT | LT]
FIELDS numfields field [field ...]
```

使用该Unix时间戳（毫秒）设置哈希中一个或多个字段的有效期。

> 可选参数的含义同HEXPIRE



示例：

```shell
redis> HSET mykey field1 "hello" field2 "world"
(integer) 2
redis> HPEXPIREAT mykey 1715704971000 FIELDS 2 field1 field2
1) (integer) 1
2) (integer) 1
redis> HPTTL mykey FIELDS 2 field1 field2
1) (integer) 303340
2) (integer) 303340
```



##### 2.4.2.5 HPERSIST

语法格式：

```shell
HPERSIST key FIELDS numfields field [field ...]
```

移除哈希中指定字段的有效期，使其变成持久的。



示例：

```shell
redis> HSET mykey field1 "hello" field2 "world"
(integer) 2
redis> HEXPIRE mykey 300 FIELDS 2 field1 field2
1) (integer) 1
2) (integer) 1
redis> HTTL mykey FIELDS 2 field1 field2
1) (integer) 283
2) (integer) 283
redis> HPERSIST mykey FIELDS 1 field2
1) (integer) 1
redis> HTTL mykey FIELDS 2 field1 field2
1) (integer) 268
2) (integer) -1
```



#### 2.4.3. 获取字段有效期

##### 2.4.3.1. HEXPIRETIME

语法格式：

```shell
HEXPIRETIME key FIELDS numfields field [field ...]
```

以Unix时间戳（秒）返回哈希字段的过期时间。



示例：

```shell
redis> HSET mykey field1 "hello" field2 "world"
(integer) 2
redis> HEXPIRE mykey 300  FIELDS 2 field1 field2
1) (integer) 1
2) (integer) 1
redis> HEXPIRETIME mykey FIELDS 2 field1 field2
1) (integer) 1715705914
2) (integer) 1715705914
```



##### 2.4.3.2. HPEXPIRETIME

语法格式：

```shell
HPEXPIRETIME key FIELDS numfields field [field ...]
```

 以Unix时间戳（毫秒）返回哈希字段的过期时间。



示例：

```shell
redis> HSET mykey field1 "hello" field2 "world"
(integer) 2
redis> HEXPIRE mykey 300 FIELDS 2 field1 field2
1) (integer) 1
2) (integer) 1
redis> HPEXPIRETIME mykey FIELDS 2 field1 field2
1) (integer) 1715705913659
2) (integer) 1715705913659
```



##### 2.4.3.3. HPTTL

语法格式：

```shell
HPTTL key FIELDS numfields field [field ...]
```

返回哈希字段的TTL（毫秒）。



示例：

```shell
redis> HPTTL no-key FIELDS 3 field1 field2 field3
(nil)
redis> HSET mykey field1 "hello" field2 "world"
(integer) 2
redis> HEXPIRE mykey 300 FIELDS 2 field1 field3
1) (integer) 1
2) (integer) -2
redis> HPTTL mykey FIELDS 3 field1 field2 field3
1) (integer) 292202
2) (integer) -1
3) (integer) -2
```



##### 2.4.3.4. HTTL

语法格式：

```shell
HTTL key FIELDS numfields field [field ...]
```

返回哈希字段的TTL（秒）。



示例：

```shell
redis> HTTL no-key FIELDS 3 field1 field2 field3
(nil)
redis> HSET mykey field1 "hello" field2 "world"
(integer) 2
redis> HEXPIRE mykey 300 FIELDS 2 field1 field3
1) (integer) 1
2) (integer) -2
redis> HTTL mykey FIELDS 3 field1 field2 field3
1) (integer) 283
2) (integer) -1
3) (integer) -2
```

