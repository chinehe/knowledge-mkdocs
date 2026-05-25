Redis 列表数据类型是简单的字符串链表，常用于实现栈和队列。

本文介绍列表数据类型的基础概念、主要特性、常用命令等内容。

## 1. 类型概述

参考官方文档：[Redis Lists](https://redis.io/docs/latest/develop/data-types/lists/)

Redis Lists列表是字符串值的链表。

列表类型常用于：

1. 实现栈和队列。
2. 为后台工作系统构建队列管理。

Redis Lists是简单的字符串列表，按照插入顺序排序。你可以从列表的**两端**添加元素（左边或右边），也可以从列表中获取、删除或操作元素。

> 可以将Redis Lists想象成排队买票的队列。新来的人（元素）可以从队尾加入（RPUSH），也可以从队头加入（LPUSH）；办理完业务的人可以从队头离开（LPOP），也可以从队尾离开（RPOP）。这种灵活性使得Lists非常适合实现各种队列和栈结构。

---

**核心特性：**

* **结构**：简单字符串值的链表。
* **长度限制**：最大长度为 2^32 - 1（4,294,967,295）个元素。
* **操作性能**：
  * 访问头和尾元素的性能很高，为O(1)。
  * 操作列表元素的命令通常为O(n)，比如`LINDEX`、`LINSERT`、`LSET`命令。执行这些命令是需要注意性能问题，特别是处理大型列表时。

* **底层实现：**在元素数量较少时使用ziplist（压缩列表），元素数量较多时使用linkedlist（链表）

> Redis 3.2版本后，Lists的底层实现改为quicklist，它是ziplist和linkedlist的结合体，既保留了ziplist的内存效率，又具备linkedlist的插入删除效率。

---

**底层实现：**

关于Redis列表的底层实现，可以参考这篇文档：参考官方文档：[Redis数据结构：List类型全面解析](https://cloud.tencent.com/developer/article/2343211)

## 2. 列表命令详解

Redis List数据类型的全部命令，可以查看官方文档：[Redis List Commands](https://redis.io/docs/latest/commands/?group=list)

### 2.1. 梳理命令列表

List相关的命令很多，但可以根据其功能按照多种方方式分类。

**按照操作方向分：**

可以分成左侧操作命令和右侧操作命令（命令中的`L`和`R`），对应操作元素的方向。比如LPUSH命令，从左侧向列表中添加元素；RPUSH命令则是从右侧向列表中添加元素。

**按照是否阻塞分：**

命令第一个字符是`B`的命令，表示阻塞命令，在有可用元素之前，该命令会被阻塞。比如BLPOP命令会被阻塞，直到列表中有元素可用。



**命令列表：**

* **基础命令**
  * **PUSH元素**
    * `LPUSH`：向列表中添加一个或多个元素。
    * `LPUSHX`：向列表中添加一个或多个元素，只有列表存在时才执行。
    * `RPUSH`：向列表中添加一个或多个元素。
    * `RPUSHX`：向列表中添加一个或多个元素，只有列表存在时才执行。
  * **POP元素**
    * `LPOP`：从列表中弹出第一个元素。
    * `LMPOP`：从多个列表中弹出第一个元素。
    * `RPOP`：弹出列表的最后一个元素。
  * **移动元素**
    * `LMOVE`：从列表中弹出一个元素，推送到另一个列表，并返回该元素。
    * `RPOPLPUSH`：从列表中弹出最后一个元素元素，推送到另一个列表，并返回该元素。（已弃用）
  * **访问元素**
    * `LPOS`：返回列表中匹配元素的索引。
    * `LINDEX`：通过索引返回列表中元素。
    * `LRANGE`：获取列表中指定范围的元素。
  * **其他基础命令**
    * `LLEN`：获取列表的长度。
    * `LREM`：从列表中移除元素。
    * `LSET`：设置列表中指定索引的元素值。
    * `LTRIM`：从列表两端移除元素。
    * `LINSERT`：在一个元素的前面或后面插入一个元素。
* **阻塞命令**
  * `BLMOVE`：从列表中弹出一个元素，推送到另一个列表，并返回该元素。
  * `BLMPOP`：从多个列表中弹出第一个元素。
  * `BLPOP`：从列表中弹出（删除并返回）第一个元素。
  * `BRPOP`：从列表中弹出最后一个元素。
  * `BRPOPLPUSH`：从列表中弹出第一个元素，推送到另一个列表，并返回该元素。（已弃用）



### 2.2. 基础操作命令

#### 2.2.1. PUSH元素

PUSH元素命令用于向列表中添加元素。

---

##### 2.2.1.1. LPUSH命令

语法格式：

```shell
LPUSH key element [element ...]
```

从左侧向列表中添加一个或多个元素。

> 从 Redis 2.4.0 版本开始接受多个`element`参数。

示例：

```shell
redis> LPUSH mylist "world"
(integer) 1
redis> LPUSH mylist "hello"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "world"
```



---

##### 2.2.1.2. LPUSHX命令

语法格式：

```shell
LPUSHX key element [element ...]
```

从左侧向列表中添加一个或多个元素，只有列表存在时才执行。

> 从 Redis 4.0.0 版本开始接受多个`element`参数。

示例：

```shell
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSHX mylist "Hello"
(integer) 2
redis> LPUSHX myotherlist "Hello"
(integer) 0
redis> LRANGE mylist 0 -1
1) "Hello"
2) "World"
redis> LRANGE myotherlist 0 -1
(empty array)
redis> 
```



---

##### 2.2.1.3. RPUSH命令

语法格式：

```shell
RPUSH key element [element ...]
```

从右侧向列表中添加一个或多个元素。



示例：

```shell
redis> RPUSH mylist "hello"
(integer) 1
redis> RPUSH mylist "world"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "world"
```



---

##### 2.2.1.4. RPUSHX命令

语法格式：

```shell
RPUSHX key element [element ...]
```

从右侧向列表中添加一个或多个元素，只有列表存在时才执行。



示例：

```shell
redis> RPUSH mylist "Hello"
(integer) 1
redis> RPUSHX mylist "World"
(integer) 2
redis> RPUSHX myotherlist "World"
(integer) 0
redis> LRANGE mylist 0 -1
1) "Hello"
2) "World"
redis> LRANGE myotherlist 0 -1
(empty array)
redis> 
```

#### 2.2.2. POP元素

POP元素命令用于从列表中弹出元素。

> 所谓弹出：移除并返回元素。

---

##### 2.2.2.1. LPOP命令

语法格式：

```shell
LPOP key [count]
```

从列表中弹出第一个元素（也就是左侧第一个元素）。

可选参数`count`用于指定要弹出的元素数量，默认为1。



示例：

```shell
redis> RPUSH mylist "one" "two" "three" "four" "five"
(integer) 5
redis> LPOP mylist
"one"
redis> LPOP mylist 2
1) "two"
2) "three"
redis> LRANGE mylist 0 -1
1) "four"
2) "five"
```



---

##### 2.2.2.2. LMPOP命令

语法格式：

```shell
LMPOP numkeys key [key ...] <LEFT | RIGHT> [COUNT count]
```

从多个列表中弹出第一个元素。元素会根据传递的列表参数，从第一个非空列表的左侧或右侧弹出。弹出元素的数量小于等于非空列表长度和计数参数（默认为1）。

* numkeys 参数用于指定key的数量。
* `<LEFT | RIGHT>` 参数用于控制弹出的方向，默认为LEFT。
* `COUNT count` 参数用于指定弹出的元素数量。



示例：

```shell
redis> LMPOP 2 non1 non2 LEFT COUNT 10
(nil)
redis> LPUSH mylist "one" "two" "three" "four" "five"
(integer) 5
redis> LMPOP 1 mylist LEFT
1) "mylist"
2) 1) "five"
redis> LRANGE mylist 0 -1
1) "four"
2) "three"
3) "two"
4) "one"
redis> LMPOP 1 mylist RIGHT COUNT 10
1) "mylist"
2) 1) "one"
   2) "two"
   3) "three"
   4) "four"
redis> LPUSH mylist "one" "two" "three" "four" "five"
(integer) 5
redis> LPUSH mylist2 "a" "b" "c" "d" "e"
(integer) 5
redis> LMPOP 2 mylist mylist2 right count 3
1) "mylist"
2) 1) "one"
   2) "two"
   3) "three"
redis> LRANGE mylist 0 -1
1) "five"
2) "four"
redis> LMPOP 2 mylist mylist2 right count 5
1) "mylist"
2) 1) "four"
   2) "five"
redis> LMPOP 2 mylist mylist2 right count 10
1) "mylist2"
2) 1) "a"
   2) "b"
   3) "c"
   4) "d"
   5) "e"
redis> EXISTS mylist mylist2
(integer) 0
redis>
```



---

##### 2.2.2.3. RPOP命令

语法格式：

```shell
RPOP key [count]
```

弹出列表的最后一个元素（也就是右侧第一个元素）。

可选参数`count`用于指定要弹出的元素数量，默认为1。



示例：

```shell
redis> RPUSH mylist "one" "two" "three" "four" "five"
(integer) 5
redis> RPOP mylist
"five"
redis> RPOP mylist 2
1) "four"
2) "three"
redis> LRANGE mylist 0 -1
1) "one"
2) "two"
```



#### 2.2.3. 移动元素

移动元素命令，从一个列表中弹出元素，并PUSH到另一个列表，最后返回该元素。

* `LMOVE`：从列表中弹出一个元素，推送到另一个列表，并返回该元素。
* `RPOPLPUSH`：从列表中弹出最后一个元素元素，推送到另一个列表，并返回该元素。（已弃用）

---

##### 2.2.3.1. LMOVE 命令

语法格式：

```shell
LMOVE source destination <LEFT | RIGHT> <LEFT | RIGHT>
```

* source 参数：源列表
* destination 参数：目标列表
* `<LEFT | RIGHT> `参数：
  * 第一个`<LEFT | RIGHT> `参数表示源列表的左侧/右侧。
  * 第二个`<LEFT | RIGHT> `参数表示目标列表的左侧/右侧。

LMOVE命令原子地从原列表的指定侧弹出元素，PUSH到目标列表的指定侧，并返回该元素。

* 如果源不存在，则不执行操作，返回nil。
* 如果源列表和目标列表相同，操作等同于从列表的指定侧移动到第二个指定侧。可将其视为列表旋转命令（如果两个指定侧不一样），或视为无操作（如果两个指定侧一样）。



示例：

```shell
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LMOVE mylist myotherlist RIGHT LEFT
"three"
redis> LMOVE mylist myotherlist LEFT RIGHT
"one"
redis> LRANGE mylist 0 -1
1) "two"
redis> LRANGE myotherlist 0 -1
1) "three"
2) "one"
redis>
```



#### 2.2.4. 访问元素

访问元素的命令，用于获取列表元素信息：

* `LPOS`：返回列表中匹配元素的索引。
* `LINDEX`：通过索引返回列表中元素。
* `LRANGE`：获取列表中指定范围的元素。

---

##### 2.2.4.1. LPOS命令

语法格式：

```shell
LPOS key element [RANK rank] [COUNT num-matches] [MAXLEN len]
```

该命令返回列表中指定元素的索引。

默认情况下，会从头到尾扫描整个列表，寻找第一个匹配的元素。如果找到该元素，则返回其索引（以0开始）。如果没有找到，就返回nil。

```shell
## 默认示例
> RPUSH mylist a b c 1 2 3 c c
> LPOS mylist c
2
```



**可选参数 RANK**：

RANK参数指定第一个要返回元素的等级，以防有多个匹配。为1表示返回第一个匹配，为2表示返回第二个匹配，以此类推。

示例：

```shell
## RANK 参数示例
> RPUSH mylist a b c 1 2 3 c c
> LPOS mylist c RANK 2
6
```



RANK参数如果为负数，则表示从尾部向头部反向搜索。

示例：

```shell
## RANK 参数示例
> RPUSH mylist a b c 1 2 3 c c
> LPOS mylist c RANK -1
7
```



**可选参数COUNT：**

有时我们不仅要返回第 N 个匹配元素，还要返回所有前 N 个匹配元素的位置。这可以通过 `COUNT` 选项实现。

```shell
## 示例1
> RPUSH mylist a b c 1 2 3 c c
> LPOS mylist c COUNT 2
[2,6]
```

我们可以结合 `COUNT` 和 `RANK`:

```shell
## 示例2
> RPUSH mylist a b c 1 2 3 c c
> LPOS mylist c RANK -1 COUNT 2
[7,6]
```

当使用 `COUNT` 时，可以指定 0 作为匹配数量，以此告诉命令我们希望所有匹配以索引数组的形式返回。这比给出很大的 `COUNT` 选项更好，因为它更通用。

```shell
> LPOS mylist c COUNT 0
[2,6,7]
```

当使用 `COUNT` 且未找到匹配时，返回一个空数组。但当未使用 `COUNT` 且无匹配时，命令返回`为零 `。



**可选参数MAXLEN：**

这个参数有助于限制命令的最大复杂度。当我们觉得很快就能找到目标值，又不想花费太多时间来确保他不匹配时，这很有用。

当使用 `MAXLEN` 时，可以指定 0 作为最大比较次数，以此告诉命令我们希望无限次比较。这比给出非常大的 `MAXLEN` 选项更好，因为它更通用。



---

##### 2.2.4.2. LINDEX命令

语法格式：

```shell
LINDEX key index
```

获取列表中指定索引的元素，索引从0开始计算。

* 如果index为负数，标识从列表尾部开始计算，如-1表示倒数第一个元素。



示例：

```shell
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LINDEX mylist 0
"Hello"
redis> LINDEX mylist -1
"World"
redis> LINDEX mylist 3
(nil)
redis>
```



---

##### 2.2.4.3. LRANGE命令

语法格式：

```shell
LRANGE key start stop
```

返回列表中从start到stop范围的元素。偏移量从0开始计算；如果为负数表示从后往前计算，-1表示倒数第一个元素。

> 超出范围的索引不会产生错误。如果 `start` 大于 列表的末尾，则返回一个空列表。如果`stop`比列表的实际末尾大，Redis 会把它当作列表的最后一个元素来处理。

示例：

```shell
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LRANGE mylist 0 0
1) "one"
redis> LRANGE mylist -3 2
1) "one"
2) "two"
3) "three"
redis> LRANGE mylist -100 100
1) "one"
2) "two"
3) "three"
redis> LRANGE mylist 5 10
(empty array)
```



#### 2.2.5. 其他基础命令

还有其他的基础命令如下：

* `LLEN`：获取列表的长度。
* `LREM`：从列表中移除元素。
* `LSET`：设置列表中指定索引的元素值。
* `LTRIM`：从列表两端移除元素。
* `LINSERT`：在一个元素的前面或后面插入一个元素。

---

##### 2.2.5.1. LLEN命令

语法格式：

```shell
LLEN key
```

返回列表的长度。

* 如果列表不存在，则返回0.
* 如果对应值不是列表，则返回错误。

示例：

```shell
redis> LPUSH mylist "World"
(integer) 1
redis> LPUSH mylist "Hello"
(integer) 2
redis> LLEN mylist
(integer) 2
```



---

##### 2.2.5.2. LREM命令

语法格式：

```shell
LREM key count element
```

从列表中移除指定的元素。count参数用于指定移除的个数：

* count大于0：从头到尾顺序移除。
* count小于0：从尾到头倒序移除。
* count等于0：移除所有等于指定值的元素。

示例：

```shell
redis> RPUSH mylist "hello"
(integer) 1
redis> RPUSH mylist "hello"
(integer) 2
redis> RPUSH mylist "foo"
(integer) 3
redis> RPUSH mylist "hello"
(integer) 4
redis> LREM mylist -2 "hello"
(integer) 2
redis> LRANGE mylist 0 -1
1) "hello"
2) "foo"
redis>
```



---

##### 2.2.5.3. LSET命令

语法格式：

```shell
LSET key index element
```

将列表中指定索引的元素设置为指定的值，对于超出范围的索引会。



示例：

```shell
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LSET mylist 0 "four"
"OK"
redis> LSET mylist -2 "five"
"OK"
redis> LRANGE mylist 0 -1
1) "four"
2) "five"
3) "three"
redis>
```



---

##### 2.2.5.4. LTRIM命令

语法格式：

```shell
LTRIM key start stop
```

修剪列表，使其只包含指定范围的元素。偏移量从0开始计算，如果为负数表示逆序。

偏移量超出列表范围时不会报错：

* 如果start大于end，或者start大于列表范围，将返回一个空列表（移除所有）。
* 如果end大于列表范围，将被当成列表末尾处理。



示例：

```shell
redis> RPUSH mylist "one"
(integer) 1
redis> RPUSH mylist "two"
(integer) 2
redis> RPUSH mylist "three"
(integer) 3
redis> LTRIM mylist 1 -1
"OK"
redis> LRANGE mylist 0 -1
1) "two"
2) "three"
redis>
```





---

##### 2.2.5.5. LINSERT命令

语法格式：

```shell
LINSERT key <BEFORE | AFTER> pivot element
```

在列表中pivot值的前面或后面（`<BEFORE | AFTER>`）插入指定的元素:

* 如果列表不存在，不执行操作。
* 如果列表存在，但是不包含pivot值，返回错误。



示例：

```shell
redis> RPUSH mylist "Hello"
(integer) 1
redis> RPUSH mylist "World"
(integer) 2
redis> LINSERT mylist BEFORE "World" "There"
(integer) 3
redis> LRANGE mylist 0 -1
1) "Hello"
2) "There"
3) "World"
redis>
```



### 2.3. 阻塞式操作命令

#### 2.3.1. 关于阻塞式操作

列表的阻塞式操作，使其非常适合用于实现队列。通常作为进程间通信系统的构建模块。下面我们简单介绍何为阻塞式操作。



想象一下，你要用列表来实现一个简单的生产者/消费者模式，这很简单：

* 生产者进程：使用LPUSH命令将消息（元素）推入消息列表。
* 消费者进程：使用RPOP命令从消息列表中获取消息。



那么问题来了，消费者并不知道消息列表中有无数据。所以它只能每隔一段时间取一次数据，如果取到了就处理数据，没有取到数据就不执行处理。这种方式叫做：**轮询**。

显然，轮询方式会强制Redis和客户端处理一些无效的命令，这会浪费一定的性能。另外轮询机制小，消费者获取消息也不是实时的，消息延迟取决于轮询间隔。



针对上述的问题，Redis提供了阻塞式操作命令。当源列表为空时，这些阻塞式操作命令会阻塞，直到超时或新元素到达才会返回给调用者。



#### 2.3.2. 命令列表

一个阻塞式操作命令是某个基础命令的变体，其功能与对应的基础命令相同，但多了**阻塞**。

命令列表如下：

* `BLMOVE`：LMOVE命令的变体（官方文档：[BLMOVE](https://redis.io/docs/latest/commands/blmove/)）
* `BLMPOP`：LMPOP命令的变体（官方文档：[BLMPOP](https://redis.io/docs/latest/commands/blmpop/)）。
* `BLPOP`：LPOP命令的变体（官方文档：[BLPOP](https://redis.io/docs/latest/commands/blpop/)）。
* `BRPOP`：RPOP命令的变体（官方文档：[BRPOP](https://redis.io/docs/latest/commands/brpop/)）。
* `BRPOPLPUSH`：RPOPLPUSH命令的变体。（已弃用）



> 本文就不去为每个阻塞操作编写讲解啰～可以参考对应的基础命令章节，一般会多一个timeout参数用于指定超时时间。