Redis基础篇001_键(Keys)的通用操作.md
管理 Redis 中的Key：Key过期、Key扫描、修改和查询Key空间

参考官方文档：[Keys and values](https://redis.io/docs/latest/develop/using-commands/keyspace/)

## 1. 键与值

Redis是一个K-V键值对的NoSql数据库，库中的每个数据都包含两个部分：

* Key 键：数据对象的唯一标识，我们通过在命令中指定key来获取或修改对应数据对象。
* Values 值：与特定Key关联的数据对象。



键值对是Redis数据存储的基本单元，Redis中的键(Keys)是访问值(Values)的入口。



键用于唯一标识存储在Redis中的值。在Redis中，所有的键都存在于一个全局的键空间中，所有数据类型（字符串、哈希、列表、集合、有序集合等）都通过键来访问。

> 可以把Redis的键想象成一个字典的索引，通过键可以快速找到对应的值。键的命名应当具有描述性，以便于识别其用途。

## 2. 键的内容与命名规范

**键的类型：**

Redis键是字符串类型，通常是文本。

但Redis实现了二进制安全（*binary-safe* ）的键，所以你可以使用任意的字节序列作为有效键，比如一个JPEG文件，或者应用中的一个struct值。另外，空字符串在Redis中也是一个有效的键。



**键的名称：**

键通常是一个具有一定含义的文本名称。与编程语言中的变量名不同，Redis键对格式限制较少，因此带有空白或标点符号的键大多也是有效的（比：如`1st Attempt`、`% of price in $`）。

Redis不支持键的命名空间或其他的类别，所有的键都存在于一个全局的键空间中，因此必须注意避免键名称冲突。依照惯例，一般使用冒号`:`将键分成多个部分，以此来将键归纳到一个类别中。比如`person：1000`、`person：2`。

Key允许的最大大小为512MB。



**键的命名规范：**

良好的键命名规范有助于数据管理、调试和维护。以下是一些推荐的键命名规范：

1. **使用冒号分隔符**：使用冒号作为命名空间分隔符，如 `user:1000:profile`、`product:100:inventory`
2. **避免特殊字符**：虽然Redis键可以包含任意二进制数据，但建议使用字母、数字、冒号、逗号和连字符。
3. **保持键长度适中**：
   1. **Key不宜过长**：过长的键名对内存不友好，而且在数据集中查找密钥可能会需要多次昂贵的密钥比较。
   2. **Key不宜过短**：过短的键名虽然会占有较少的内存，但如果因此丧失可阅读性，显然就得不偿失了。我们的目标是找到一个合适的平衡点。比如将键从`user：1000：followers`改成`u1000flw`，显然就不合适了。

4. **语义清晰**：键名应当能清晰表达其对应的值的含义。
5. **模式统一**：键的命名尽量保持一个模式。比如按照模式`object-type：id`,将键写作`user：1000`、user：1001。点或破折号常用于多词字段，比如`comment：4321：reply.to`或`comment：4321：reply-to`。

## 3. 键的操作

>  键的操作远远不止下面的这些命令，这里只是简单介绍一些常用的命令～
>
> 全量的Redis命令请参考官方文档：[Redis Commands](https://redis.io/docs/latest/commands/)

### 3.1. 基本操作

#### 3.1.1. SET 设置键

参考官方文档：[SET 命令](https://redis.io/docs/latest/commands/set/)

SET是设置Redis键最基础的命令（还有其他设置Redis键的命令），它设置一个Redis键以保存字符串值。如果之前已经有相同的键，则无论其类型如何，都会被覆盖。



命令格式：

```shell
SET key value [NX | XX | IFEQ ifeq-value | IFNE ifne-value |
  IFDEQ ifdeq-digest | IFDNE ifdne-digest] [GET] [EX seconds |
  PX milliseconds | EXAT unix-time-seconds |
  PXAT unix-time-milliseconds | KEEPTTL]
```



示例：

```shell
redis> SET mykey "Hello"
"OK"
redis> GET mykey
"Hello"
redis> SET anotherkey "will expire in a minute" EX 60
"OK"
```



#### 3.1.2. GET 获取键

参考官方文档：[GET 命令](https://redis.io/docs/latest/commands/get/)

GET是获取Redis键所存储值（只处理字符串值）最基础的命令。如果键不存在，则返回特殊值nil，如果键所存储值不是字符串，则会返回错误。



命令格式：

```shell
GET key
```



示例：

```shell
redis> GET nonexisting
(nil)
redis> SET mykey "Hello"
"OK"
redis> GET mykey
"Hello"
```



#### 3.1.3. TYPE 获取键值的类型

参考官方文档：[TYPE 命令](https://redis.io/docs/latest/commands/type/)

TYPE命令用于获取指定键所存储值类型的字符串标识，返回不同的类型包括：`string`, `list`, `set`, `zset`, `hash`, `stream`, and `vectorset`.



命令格式：

```shell
TYPE key
```



示例：

```shell
redis> SET key1 "value"
"OK"
redis> LPUSH key2 "value"
(integer) 1
redis> SADD key3 "value"
(integer) 1
redis> TYPE key1
"string"
redis> TYPE key2
"list"
redis> TYPE key3
"set"
```



#### 3.1.4. EXISTS 检查键是否存在

参考官方文档：[EXISTS 命令](https://redis.io/docs/latest/commands/exists/)

TYPE命令用于检查指定键是否存在。其参数可以指定多个Key，返回值为这些Key存在的数量。

>  如果一个存在的Key在参数列表中多次出现，会被多次计数。



命令格式：

```shell
EXISTS key [key ...]
```



示例：

```shell
redis> SET key1 "Hello"
"OK"
redis> EXISTS key1
(integer) 1
redis> EXISTS nosuchkey
(integer) 0
redis> SET key2 "World"
"OK"
redis> EXISTS key1 key2 nosuchkey
(integer) 2
```



#### 3.1.5. DEL删除键

参考官方文档：[DEL 命令](https://redis.io/docs/latest/commands/del/)

DEL命令移除指定的Key。如果Key不存在，则该Key被忽略。



命令格式：

```shell
DEL key [key ...]
```



示例：

```shell
redis> SET key1 "Hello"
"OK"
redis> SET key2 "World"
"OK"
redis> DEL key1 key2 key3
(integer) 2
```

### 3.2. SCAN 键的扫描

参考官方文档：[SCAN 命令](https://redis.io/docs/latest/commands/scan/)



SCAN是一个基于游标的迭代命令，用于增量地遍历键空间，相比KEYS命令更安全，不会阻塞服务器。

语法：
```
SCAN cursor [MATCH pattern] [COUNT count] [TYPE type]
```

- cursor: 游标值，首次调用使用0
- MATCH: 可选，匹配模式
- COUNT: 可选，每次迭代返回的元素数量估计
- TYPE: 可选，限制返回特定类型的键

示例：
```
redis> scan 0 match user:*
1) "28"
2) 1) "user:1000"
   2) "user:1001"
   3) "user:1002"
redis> scan 28
1) "0"
2) 1) "user:1003"
   2) "user:1004"
```

### 3.3. KEYS 遍历键

参考官方文档：[KEYS 命令](https://redis.io/docs/latest/commands/keys/)

虽然SCAN是推荐的方式，但KEYS命令在小数据集上仍有用处。它返回所有匹配给定模式的键。

> **注意：此命令在大数据集上使用会阻塞服务器，生产环境中应避免使用。**

模式匹配规则：
- `?` 匹配单个字符
- `*` 匹配零个或多个字符
- `[...]` 匹配方括号内的任意字符
- `\x` 转义字符x

示例：
```
redis> SET user:1000 "Alice"
OK
redis> SET user:1001 "Bob"
OK
redis> SET product:100 "iPhone"
OK
redis> KEYS user:*
1) "user:1000"
2) "user:1001"
redis> KEYS *
1) "user:1000"
2) "user:1001"
3) "product:100"
```

### 3.4. 键的重命名

键的重命名有两个命令：

* RENAME 命令：参考官方文档：[RENAME 命令](https://redis.io/docs/latest/commands/rename/)。
* RENAMENX 命令：参考官方文档：[RENAMENX 命令](https://redis.io/docs/latest/commands/renamenx/)

------

**RENAME命令：**

RENAME命令将指定的Key重命名为新的Key。

* 如果原Key不存在，返回错误。
* 如果新Key已经存在，则会被覆盖。此时RENAME命令会执行隐式的DEL操作，因此如果删除的Key包含很大的值，即使RENAME操作本身使用常量时间，也可能引起较高的延迟。
* 在集群模式下，新旧Key必须处于同一个哈希槽中，也就是说实际上只有具有相同hash tag的key才能在集群中被可靠地重命名。



命令格式：

```shell
RENAME key newkey
```

示例：

```shell
redis> SET mykey "Hello"
"OK"
redis> RENAME mykey myotherkey
"OK"
redis> GET myotherkey
"Hello"
```



-----

**RENAMENX命令：**

如果新Key不存在，RENAME命令将指定的Key重命名为新的Key。

如果新Key已经存在，则命令不会执行操作。（这是与RENAME命令区别所在）



命令格式：

```shell
RENAMENX key newkey
```

示例：

```shell
redis> SET mykey "Hello"
"OK"
redis> SET myotherkey "World"
"OK"
redis> RENAMENX mykey myotherkey
(integer) 0
redis> GET myotherkey
"World"
```



## 4. 键的过期时间

### 4.1. key expiration概述

Redis还有一个重要的特性：**key expiration 键过期**。

Redis允许你为键设置超时时间（也叫存活时间、TTL），当存活时间结束（过期）时，键会被自动销毁。

> 💡 **初学者小贴士**：可以把Redis的键过期功能想象成一个定时器，当设置一个键的过期时间后，Redis会在后台启动一个定时器，时间到了就自动删除这个键。这种机制在实际开发中非常有用，比如用于缓存、会话管理、验证码存储等场景。

注意：

* 可以使用秒级或毫秒级精度设置过期时间，但过期时间的分辨率始终是1毫秒。

  > Redis 内部始终以 1 毫秒为最小单位进行过期时间计算。无论使用秒级还是毫秒级设置，实际的过期检查和删除操作都以毫秒为精度执行。
  >
  > 这种设计既保持了 API 的灵活性（支持不同精度的设置方式），又确保了内部处理的一致性和精确性。

### 4.2. 设置过期时间

Redis提供了多种方式来设置键的过期时间，每种方式适用于不同的场景：

#### 4.2.1. EXPIRE 命令

参考官方文档：[EXPIRE 命令](https://redis.io/docs/latest/commands/expire/)

EXPIRE命令用于为已存在的键设置过期时间（以秒为单位）。如果键存在，则设置成功返回1，如果键不存在则返回0。

命令格式：
```
EXPIRE key seconds
```

示例：
```
redis> SET mykey "Hello"
OK
redis> EXPIRE mykey 10
(integer) 1
redis> TTL mykey
(integer) 9
```

#### 4.2.2. PEXPIRE 命令

参考官方文档：[PEXPIRE 命令](https://redis.io/docs/latest/commands/pexpire/)

PEXPIRE命令与EXPIRE命令类似，但以毫秒为单位设置过期时间。如果键存在，则设置成功返回1，如果键不存在则返回0。

命令格式：
```
PEXPIRE key milliseconds
```

示例：
```
redis> SET mykey "Hello"
OK
redis> PEXPIRE mykey 5000
(integer) 1
redis> PTTL mykey
(integer) 4999
```

#### 4.2.3. EXPIREAT 命令

参考官方文档：[EXPIREAT 命令](https://redis.io/docs/latest/commands/expireat/)

EXPIREAT命令用于为已存在的键设置过期时间戳（以秒为单位的Unix时间戳）。当Redis服务器的当前时间大于等于设置的时间戳时，键将被删除。

命令格式：
```
EXPIREAT key unix-time-seconds
```

示例：
```
redis> SET mykey "Hello"
OK
redis> EXPIREAT mykey 1672531200  # 2023年1月1日的时间戳
(integer) 1
```

#### 4.2.4. PEXPIREAT 命令

参考官方文档：[PEXPIREAT 命令](https://redis.io/docs/latest/commands/pexpireat/)

PEXPIREAT命令与EXPIREAT命令类似，但使用毫秒级Unix时间戳作为过期时间。

命令格式：
```
PEXPIREAT key unix-time-milliseconds
```

示例：
```
redis> SET mykey "Hello"
OK
redis> PEXPIREAT mykey 1672531200000  # 2023年1月1日的毫秒时间戳
(integer) 1
```

#### 4.2.5. SET命令中的过期选项

在SET命令中可以直接设置过期时间，这种方式可以原子性地设置键值对和过期时间，避免了两次操作的开销。

- `EX seconds`：设置键的过期时间（秒）
- `PX milliseconds`：设置键的过期时间（毫秒）
- `EXAT timestamp`：设置键的过期时间戳（秒）
- `PXAT timestamp`：设置键的过期时间戳（毫秒）

示例：
```
redis> SET mykey "Hello" EX 60
OK
redis> SET anotherkey "World" PX 5000
OK
redis> SET sessionid "123456" EXAT 1672531200
OK
```

### 4.3. PERSIST 移除过期时间

参考官方文档：[PERSIST 命令](https://redis.io/docs/latest/commands/persist/)

PERSIST命令用于移除键的过期时间，使键变为永久有效（永不过期）。如果键存在且设置了过期时间，则移除成功返回1；如果键不存在或没有设置过期时间，则返回0。

命令格式：
```
PERSIST key
```

示例：
```
redis> SET mykey "Hello"
OK
redis> EXPIRE mykey 60
(integer) 1
redis> TTL mykey
(integer) 59
redis> PERSIST mykey
(integer) 1
redis> TTL mykey
(integer) -1
```


### 4.4. 检查过期时间

#### 4.4.1. TTL 命令

参考官方文档：[TTL 命令](https://redis.io/docs/latest/commands/ttl/)

TTL命令用于获取键的剩余生存时间（Time To Live），以秒为单位。返回值含义如下：

- 正整数：键还有多少秒过期
- -1：键存在但没有设置过期时间
- -2：键不存在

命令格式：
```
TTL key
```

示例：
```
redis> SET mykey "Hello"
OK
redis> EXPIRE mykey 60
(integer) 1
redis> TTL mykey
(integer) 59
redis> PERSIST mykey
(integer) 1
redis> TTL mykey
(integer) -1
redis> TTL nonexisting
(integer) -2
```

#### 4.4.2. PTTL 命令

参考官方文档：[PTTL 命令](https://redis.io/docs/latest/commands/pttl/)

PTTL命令与TTL命令类似，但返回剩余生存时间以毫秒为单位。返回值含义如下：

- 正整数：键还有多少毫秒过期
- -1：键存在但没有设置过期时间
- -2：键不存在

命令格式：
```
PTTL key
```

示例：
```
redis> SET mykey "Hello"
OK
redis> PEXPIRE mykey 5000
(integer) 1
redis> PTTL mykey
(integer) 4999
redis> PERSIST mykey
(integer) 1
redis> PTTL mykey
(integer) -1
redis> PTTL nonexisting
(integer) -2
```

### 4.5. 过期键的删除策略

Redis使用两种策略来删除过期键：

1. **惰性删除**：当访问一个键时，才检查该键是否过期，如果过期则删除。
2. **定期删除**：每隔一段时间，Redis会随机检查一部分设置了过期时间的键，删除其中已经过期的键。

这种混合策略在CPU使用率和内存使用率之间做了平衡，既不会因为定时删除所有过期键而消耗过多CPU资源，也不会因为只使用惰性删除而浪费过多内存。

> 由于Redis使用的是定期删除和惰性删除的混合策略，所以一个键的过期时间并不是绝对精确的。在某些情况下，过期的键可能在过期时间之后仍然存在于内存中，直到惰性删除或定期删除机制触发为止。