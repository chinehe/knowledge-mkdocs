Redis Sets集合是一个无序的唯一字符串成员的集合，可以利用集合来高效地跟踪唯一项目、表示关系、以及执行集合运算等。

文章首先简单讲解Sets集合类型的一些概念和特性；后面重点讲解Sets类型相关的命令，包括所有命令的语法、参数，并提供示例。

参考官方文档：[Redis Sets](https://redis.io/docs/latest/develop/data-types/sets/)

## 1. 类型概述

Redis Sets（集合）是一个无序且唯一的字符串集合。你可以添加、删除和检查成员的存在性，还可以执行多个集合间的操作，如并集、交集和差集。

可以利用Sets来高效地实现：

* 跟踪唯一项目：比如记录访问某篇博客的所有唯一IP地址。
* 表示关系：比如所有具有特定角色的用户集合。
* 执行常见的集合运算：比如交集、并集、差集。



**核心特性：**

* **结构：**无序的唯一字符串集合
* 容量**限制： **Redis 集的最大容量为 2^32 - 1 个（4,294,967,295）个成员。
* **性能： **大多数集合操作，比如添加、删除和检查某个元素，都是O(1)，效率很高。而`SMEMBERS`命令是O(n)，因此在大集合上面执行该命令时需要谨慎，可以考虑使用`SSCAN`作为替代方案。
* **底层实现：**当集合元素数量较少时使用intset（整数集合），否则使用hashtable（哈希表）

> Redis会根据集合中元素的特点自动选择最合适的编码方式。如果所有元素都是整数且数量不多，会使用更节省内存的intset编码；否则使用hashtable编码。

## 2. 命令详解

参考官方文档：[Commands of sets](https://redis.io/docs/latest/commands/?group=set)

### 2.1. 命令列表

* 操作成员
  * `SADD`：向集合中添加一个或多个成员。
  * `SMOVE`：将集合中的一个成员移动到另一个集合。
  * `SPOP`：从集合中随机弹出一个或多个成员。
  * `SREM`：从集合中移除一个或多个成员。
* 访问成员
  * `SCARD`：返回集合中的成员数量。
  * `SISMEMBER`：检查元素是否为集合的成员。
  * `SMEMBERS`：返回集合的所有成员。
  * `SMISMEMBER`：检查多个元素是否为集合的成员。
  * `SSCAN`：对集合中的成员进行迭代。
  * `SRANDMEMBER`：从集合中返回一个或多个成员。

* 集合运算
  * `SDIFF`：返回多个集合的差集。
  * `SDIFFSTORE`：计算多个集合的差集并存储到指定的键中。
  * `SINTER`：计算多个集合的交集。
  * `SINTERCARD`：计算多个集合的交集的成员数量。
  * `SINTERSTORE`：计算多个集合的交集并存储到指定的键中。
  * `SUNION`：计算多个集合的并集。
  * `SUNIONSTORE`：计算多个集合的并集并存储到指定的键中。

### 2.2. 操作成员

操作成员命令用于操作集合中的成员，比如添加成员、删除成员、移动成员和弹出成员。

#### 2.2.1. SADD

命令语法：

```shell
SADD key member [member ...]
```

向集合中添加一个或多个成员,如果成员已存在，则忽略。



示例：

```shell
redis> SADD myset "Hello" "World"
(integer) 2
redis> SADD myset "World"
(integer) 0
redis> SMEMBERS myset
1) "Hello"
2) "World"
```

#### 2.2.2. SMOVE

命令语法：

```shell
SMOVE source destination member
```

将集合中的一个成员移动到另一个集合。



示例：

```shell
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myotherset "three"
(integer) 1
redis> SMOVE myset myotherset "two"
(integer) 1
redis> SMEMBERS myset
1) "one"
redis> SMEMBERS myotherset
1) "three"
2) "two"
```

#### 2.2.3. SPOP

命令语法：

```shell
SPOP key [count]
```

从集合中随机弹出一个或多个成员。

默认情况下，命令会从集合中弹出一个成员。当提供可选的count参数时，将最多弹出count个成员，实际数量取决于集合中的成员数量。



示例：

```shell
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myset "three"
(integer) 1
redis> SPOP myset
"three"
redis> SMEMBERS myset
1) "one"
2) "two"
redis> SADD myset "four"
(integer) 1
```



#### 2.2.4. SREM

命令语法：

```shell
SREM key member [member ...]
```

从集合中移除一个或多个成员。



示例：

```shell
redis> SADD myset "one"
(integer) 1
redis> SADD myset "two"
(integer) 1
redis> SADD myset "three"
(integer) 1
redis> SREM myset "one"
(integer) 1
redis> SREM myset "four"
(integer) 0
redis> SMEMBERS myset
1) "two"
2) "three"
redis>
```



### 2.3. 访问成员

访问成员命令用户获取集合中的成员或成员信息，比如检查成员是否存在、获取集合中的成员。

#### 2.3.1. SCARD

命令语法：

```shell
SCARD key
```

返回集合中的成员数量。



示例：

```shell
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SCARD myset
(integer) 2
redis>
```



#### 2.3.2. SISMEMBER

命令语法：

```shell
SISMEMBER key member
```

检查元素是否为集合的成员。



示例：

```shell
redis> SADD myset "one"
(integer) 1
redis> SISMEMBER myset "one"
(integer) 1
redis> SISMEMBER myset "two"
(integer) 0
redis> 
```



#### 2.3.3. SMEMBERS

命令语法：

```shell
SMEMBERS key
```

返回集合的所有成员。



示例：

```shell
redis> SADD myset "Hello"
(integer) 1
redis> SADD myset "World"
(integer) 1
redis> SMEMBERS myset
1) "Hello"
2) "World"
```



#### 2.3.4. SMISMEMBER

命令语法：

```shell
SMISMEMBER key member [member ...]
```

检查多个元素是否为集合的成员。



示例：

```shell
redis> SADD myset "one"
(integer) 1
redis> SADD myset "one"
(integer) 0
redis> SMISMEMBER myset "one" "notamember"
1) (integer) 1
2) (integer) 0
redis> 
```



#### 2.3.5. SSCAN

命令语法：

```shell
SSCAN key cursor [MATCH pattern] [COUNT count]
```

对集合中的成员进行迭代。



请参考官方文档：[Redis Command Scan](https://redis.io/docs/latest/commands/scan/)



#### 2.3.6. SRANDMEMBER

命令语法：

```shell
SRANDMEMBER key [count]
```

从集合中返回一个或多个成员。

* 当count参数为正数时，表示最多返回的成员个数。实际返回个数取count 与集合成员数的较低值。
* 当count参数为负数时，命令返回的成员可以重复，实际返回个数为count的绝对值。



示例：

```shell
redis> SADD myset one two three
(integer) 3
redis> SRANDMEMBER myset
"one"
redis> SRANDMEMBER myset 2
1) "one"
2) "three"
redis> SRANDMEMBER myset -5
1) "three"
2) "two"
3) "two"
4) "two"
5) "three"
redis> 
```



### 2.4. 集合运算

集合运算命令用于执行多个集合的集合运算，比如计算交集、并集、差集。

#### 2.4.1. SDIFF

命令语法：

```shell
SDIFF key [key ...]
```

计算多个集合的差集。



示例：

```shell
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SDIFF key1 key2
1) "a"
2) "b"
redis> 
```



#### 2.4.2. SDIFFSTORE

命令语法：

```shell
SDIFFSTORE destination key [key ...]
```

计算多个集合的差集并存储到指定的键中。



示例：

```shell
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SDIFFSTORE key key1 key2
(integer) 2
redis> SMEMBERS key
1) "a"
2) "b"
redis>
```



#### 2.4.3. SINTER

命令语法：

```shell
SINTER key [key ...]
```

计算多个集合的交集。



示例：

```shell
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SINTER key1 key2
1) "c"
```



#### 2.4.4. SINTERCARD

命令语法：

```shell
SINTERCARD numkeys key [key ...] [LIMIT limit]
```

计算多个集合的交集的成员数量。其中`numkeys`表示参与计算的键的数量（必须为整数）。

默认情况下，该命令计算所有给定集合交集的基数。当提供可选的 `LIMIT` 参数（默认为 0 且表示无限）时，如果计算过程中交集基数达到极限，算法将退出并得到基数极限。



示例：

```shell
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key1 "d"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SINTER key1 key2
1) "c"
2) "d"
redis> SINTERCARD 2 key1 key2
(integer) 2
redis> SINTERCARD 2 key1 key2 LIMIT 1
(integer) 1
redis>
```



#### 2.4.5. SINTERSTORE

命令语法：

```shell
SINTERSTORE destination key [key ...]
```

计算多个集合的交集并存储到指定的键中。



示例：

```shell
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SINTERSTORE key key1 key2
(integer) 1
redis> SMEMBERS key
1) "c"
redis> 
```



#### 2.4.6. SUNION

命令语法：

```shell
SUNION key [key ...]
```

计算多个集合的并集。



示例：

```shell
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SUNION key1 key2
1) "b"
2) "c"
3) "e"
4) "a"
5) "d"
redis> 
```



#### 2.4.7. SUNIONSTORE

命令语法：

```shell
SUNIONSTORE destination key [key ...]
```

计算多个集合的并集并存储到指定的键中。



示例：

```shell
redis> SADD key1 "a"
(integer) 1
redis> SADD key1 "b"
(integer) 1
redis> SADD key1 "c"
(integer) 1
redis> SADD key2 "c"
(integer) 1
redis> SADD key2 "d"
(integer) 1
redis> SADD key2 "e"
(integer) 1
redis> SUNIONSTORE key key1 key2
(integer) 5
redis> SMEMBERS key
1) "a"
2) "b"
3) "c"
4) "d"
5) "e"
redis> 
```