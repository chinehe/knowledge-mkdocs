Redis SortedSets有序集合是一组按照关联分数排序的唯一字符串成员的集合，当多个字符串拥有相同的分数时，这些字符串按照字典序进行排序。

文章先简要介绍了SortedSets有序集合类型的概念和特性，然后重点介绍了类型相关的所有命令，并提供了详细的示例。



参考官方文档：[Redis SortedSets](https://redis.io/docs/latest/develop/data-types/sorted-sets/)

## 1. 类型概述

Redis SortedSets有序集合是一组按照关联分数排序的唯一字符串成员的集合，当多个字符串拥有相同的分数时，这些字符串按照字典序进行排序。

有序集合的常用场景包括：

* 排行榜：比如，你可以使用有序集合轻松地维护大型在线游戏中最高分数的排序列表。
* 限速器：比如，你可以使用有序集合来实现滑动窗口限速器，以防止过多的API请求。



> 你可以把有序集合想象成集合和哈希的混合体。
>
> * 和集合一样，有序集合由唯一不重复的字符串元素组成，因此从某种意义来说，排序集也是一个集合。
> * 有序集合内每个元素都关联以恶搞浮点值（分数），这点类似于哈希，因为每个元素映射一个值。



有序集合底层采用调表（skiplist）和字典（hashtable）实现。每次添加元素是，Redis都会执行复杂度为O(log(N))的操作。这样我们需要元素排序时，就不需要再执行任何操作了，因为已经排序好了。



**核心特性：**

* **结构：** 唯一字符串及其关联分数的集合
* **性能：** 大多数有序集合的操作复杂度为O(log(n))，其中n为成员数。而`ZRANGE`命令的复杂度为O(log(n)+m)，其中m为返回的结果数量，因此在执行返回值较大的操作时，应当谨慎。
* **底层实现：**使用跳跃表（skiplist）和字典（hashtable）的组合

> 跳跃表提供了高效的范围查询和有序遍历能力，而字典保证了O(1)时间复杂度的成员查找。这种双重数据结构的设计使得Sorted Sets在保持排序的同时，也具备了快速的查找性能。

## 2. 命令详解

参考官方文档：[Redis Commands of SortedSets](https://redis.io/docs/latest/commands/?group=sorted-set)

### 2.1. 命令列表

* 基本操作
  * `ZADD`： 向有序集合中添加一个或多个成员，或者更新其分数。
  * `ZCARD`：获取有序集合中的成员数量。
  * `ZCOUNT`：获取有序集合中指定分数范围的成员数量。
  * `ZINCRBY`：将有序集合中指定的成员分数加N。
  * `ZRANK`：获取按分数升序排序的成员的排名。
  * `ZREVRANK`：获取有序集合中成员的排名（按照分数逆序）。
  * `ZREM`：从有序集合中移除一个或多个成员。
  * `ZSCAN`：迭代有序集合的成员。
  * `ZSCORE`：获取有序集合中指定成员的分数。
  * `ZMSCORE`：获取有序集合中一个或多个成员的分数。
  * `ZPOPMAX`：弹出有序集合中分数最高的成员。
  * `ZPOPMIN`：弹出有序集合中分数最低的成员。
  * `ZMPOP`：弹出一个或多个有序集合中分数最高或最低的成员。
  * `ZRANDMEMBER`：随机获取有序集合中一个或多个成员。
* 范围操作
  * `ZLEXCOUNT`：获取有序集合中指定字典序范围的成员数量。
  * `ZRANGE`：获取有序集合中指定索引范围内的成员。
  * `ZRANGEBYLEX`：获取有序集合中指定字典序范围的成员。（已弃用）
  * `ZRANGEBYSCORE`：获取有序集合中指定分数范围内的成员。（已弃用）
  * `ZRANGESTORE`：获取有序集合中指定范围的成员，并存储到指定的键中。
  * `ZREMRANGEBYLEX`：移除有序集合中指定字典序范围内的成员。
  * `ZREMRANGEBYRANK`：移除有序集合中指定排名范围内的成员。
  * `ZREMRANGEBYSCORE`：移除有序集合中指定分数范围内的成员。
  * `ZREVRANGE`：获取有序集合中指定排名范围内的成员（逆序）。（已弃用）
  * `ZREVRANGEBYLEX`：获取有序集合中指定字典序范围内的成员（逆序）。（已弃用）
  * `ZREVRANGEBYSCORE`：获取有序集合中指定分数范围内的成员（逆序）。（已弃用）
* 集合运算
  * `ZDIFF`：计算多个有序集合的差集。
  * `ZDIFFSTORE`：计算多个有序集合的差集，并将结果保存到指定的键中。
  * `ZINTER`：计算多个有序集合的交集。
  * `ZINTERCARD`：获取多个有序集合的交集的数量。
  * `ZINTERSTORE`：计算多个有序集合的交集，并将结果保存到指定的键中。
  * `ZUNION`：计算多个有序集合的并集。
  * `ZUNIONSTORE`：计算多个有序集合的并集，并将结果存储到指定的键中。
* 阻塞操作
  * `BZMPOP`：按照分数从一个或多个有序集合中弹出一个成员。
  * `BZPOPMAX`：从一个或多个有序集合中弹出分数最高的成员。
  * `BZPOPMIN`：从一个或多个有序集合中弹出分数最低的成员。

### 2.2. 基本操作

基本操作命令用于对有序集合进行基础的增删改查操作。

#### 2.2.1. ZADD

命令语法：

```shell
ZADD key [NX | XX] [GT | LT] [CH] [INCR] score member [score member ...]
```

向有序集合中添加一个或多个成员，或者更新已存在成员的分数。

分数为双精度浮点数字符串表示，+inf和-inf也是有效的值。



可选参数：

- **NX**：只添加新成员，不更新已存在成员
- **XX**：只更新已存在成员，不添加新成员（与NX互斥）
- **GT**：仅当新分数大于当前分数时更新（与LT互斥）
- **LT**：仅当新分数小于当前分数时更新（与GT互斥）
- **CH**：返回更改的数量（包括新添加和分数变更的成员）
- **INCR**：将分数增量添加到成员（只能指定一个成员）

> ZADD命令的参数组合需要特别注意互斥关系，例如NX和XX不能同时使用。当使用INCR选项时，只能操作单个成员，此时命令返回更新后的分数（字符串形式）而非操作数量。

示例：

```shell
> ZADD myzset 1 "one"
(integer) 1
> ZADD myzset 1 "uno"
(integer) 1
> ZADD myzset 2 "two" 3 "three"
(integer) 2
> ZRANGE myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "uno"
4) "1"
5) "two"
6) "2"
7) "three"
8) "3"
```

#### 2.2.2. ZCARD

命令语法：

```shell
ZCARD key
```

获取有序集合中的成员数量。



示例：

```shell
redis> ZADD myzset 1 "one" 2 "two"
(integer) 2
redis> ZCARD myzset
(integer) 2
```

#### 2.2.3. ZCOUNT

命令语法：

```shell
ZCOUNT key min max
```

获取有序集合中指定分数范围的成员数量。

- min和max可以是具体数值，也可以使用`-inf`（负无穷）和`+inf`（正无穷）
- 区间包含边界值，使用`(`表示开区间（如`(5`）

示例：

```shell
redis> ZADD myzset 1 "one" 2 "two" 3 "three"
(integer) 3
redis> ZCOUNT myzset 1 2
(integer) 2
redis> ZCOUNT myzset (1 +inf
(integer) 2
```

#### 2.2.4. ZINCRBY

命令语法：

```shell
ZINCRBY key increment member
```

将有序集合中指定的成员分数加N。

- increment可以是整数或浮点数
- 如果成员不存在，则先添加成员，分数设置为increment

示例：

```shell
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZINCRBY myzset 2 "one"
"3"
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "two"
2) "2"
3) "one"
4) "3"
redis>
```

#### 2.2.5. ZRANK

命令语法：

```shell
ZRANK key member [WITHSCORE]
```

获取指定成员的排名（升序排序）。

WITHSCORE可选参数用于返回分数。



示例：

```shell
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZRANK myzset "three"
(integer) 2
redis> ZRANK myzset "four"
(nil)
redis> ZRANK myzset "three" WITHSCORE
1) (integer) 2
2) "3"
```

#### 2.2.6. ZREVRANK

命令语法：

```shell
ZREVRANK key member [WITHSCORE]
```

获取指定成员的排名（降序排序）。

WITHSCORE可选参数用于返回分数。



示例：

```shell
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREVRANK myzset "one"
(integer) 2
redis> ZREVRANK myzset "four"
(nil)
redis> ZREVRANK myzset "three" WITHSCORE
1) (integer) 0
2) "3"
redis> ZREVRANK myzset "four" WITHSCORE
(nil)
redis>
```

#### 2.2.7. ZREM

命令语法：

```shell
ZREM key member [member ...]
```

从有序集合中移除一个或多个成员。

示例：

```shell
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREM myzset "two"
(integer) 1
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "three"
4) "3"
redis> 
```

#### 2.2.8. ZSCAN

命令语法：

```shell
ZSCAN key cursor [MATCH pattern] [COUNT count]
```

迭代有序集合的成员。

请参考官方文档：[Redis Command Scan](https://redis.io/docs/latest/commands/scan/)



#### 2.2.9. ZSCORE

命令语法：

```shell
ZSCORE key member
```

获取有序集合中指定成员的分数。



示例：

```shell
redis> ZADD myzset 1.5 "one"
(integer) 1
redis> ZSCORE myzset "one"
"1.5"
```

#### 2.2.10. ZMSCORE

命令语法：

```shell
ZMSCORE key member [member ...]
```

获取有序集合中一个或多个成员的分数（Redis 6.2+）。



示例：

```shell
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZMSCORE myzset "one" "two" "nofield"
1) "1"
2) "2"
3) (nil)
redis> 
```

#### 2.2.11. ZPOPMAX

命令语法：

```shell
ZPOPMAX key [count]
```

弹出有序集合中分数最高的成员（Redis 5.0+）。

- 默认弹出1个成员
- 当提供count参数时，最多弹出count个成员

示例：

```shell
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZPOPMAX myzset
1) "three"
2) "3"
redis> 
```

#### 2.2.12. ZPOPMIN

命令语法：

```shell
ZPOPMIN key [count]
```

弹出有序集合中分数最低的成员（Redis 5.0+）。



示例：

```shell
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZPOPMIN myzset
1) "one"
2) "1"
redis> 
```

#### 2.2.13. ZMPOP

命令语法：

```shell
ZMPOP numkeys key [key ...] [MIN | MAX] [COUNT count]
```

从一个或多个有序集合中第一个非空有序集合中弹出分数最高或最低一个或多个的成员（Redis 7.0+）。



示例：

```shell
redis> ZMPOP 1 notsuchkey MIN
(nil)
redis> ZADD myzset 1 "one" 2 "two" 3 "three"
(integer) 3
redis> ZMPOP 1 myzset MIN
1) "myzset"
2) 1) 1) "one"
      2) "1"
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "two"
2) "2"
3) "three"
4) "3"
redis> ZMPOP 1 myzset MAX COUNT 10
1) "myzset"
2) 1) 1) "three"
      2) "3"
   2) 1) "two"
      2) "2"
redis> ZADD myzset2 4 "four" 5 "five" 6 "six"
(integer) 3
redis> ZMPOP 2 myzset myzset2 MIN COUNT 10
1) "myzset2"
2) 1) 1) "four"
      2) "4"
   2) 1) "five"
      2) "5"
   3) 1) "six"
      2) "6"
redis> ZRANGE myzset 0 -1 WITHSCORES
(empty array)
redis> ZMPOP 2 myzset myzset2 MAX COUNT 10
(nil)
redis> ZRANGE myzset2 0 -1 WITHSCORES
(empty array)
redis> EXISTS myzset myzset2
(integer) 0
redis> 
```

#### 2.2.14. ZRANDMEMBER

命令语法：

```shell
ZRANDMEMBER key [count [WITHSCORES]]
```

随机获取有序集合中一个或多个成员。

- 当count参数为正数时，表示最多返回的成员个数。实际返回个数取count 与集合成员数的较低值。
- 当count参数为负数时，命令返回的成员可以重复，实际返回个数为count的绝对值。

示例：

```shell
redis> ZADD dadi 1 uno 2 due 3 tre 4 quattro 5 cinque 6 sei
(integer) 6
redis> ZRANDMEMBER dadi
"tre"
redis> ZRANDMEMBER dadi
"cinque"
redis> ZRANDMEMBER dadi -5 WITHSCORES
1) "quattro"
2) "4"
3) "tre"
4) "3"
5) "cinque"
6) "5"
7) "cinque"
8) "5"
9) "sei"
10) "6"
redis> 
```

### 2.3. 范围操作

范围操作命令用于获取有序集合中指定范围的成员。

#### 2.3.1. ZLEXCOUNT

命令语法：

```shell
ZLEXCOUNT key min max
```

获取有序集合中指定字典序范围的成员数量。

- min和max使用`[`表示包含，`(`表示不包含
- 使用`-`表示负无穷，`+`表示正无穷

示例：

```shell
redis> ZADD myzset 0 a 0 b 0 c 0 d 0 e
(integer) 5
redis> ZADD myzset 0 f 0 g
(integer) 2
redis> ZLEXCOUNT myzset - +
(integer) 7
redis> ZLEXCOUNT myzset [b [f
(integer) 5
redis>
```

#### 2.3.2. ZRANGE

命令语法：

```shell
ZRANGE key start stop 
	[BYSCORE | BYLEX]
  [REV] 
  [LIMIT offset count]
  [WITHSCORES]
```

获取有序集合中指定索引范围内的成员。



可选参数：

- 排序方式
  - BYSCORE：按分数排序
  - BYLEX：按字典排序

- REV：是否降序排序。
- LIMIT：个数限制
- WITHSCORES：返回分数

示例：

```shell
redis> ZADD myzset 1 "one" 2 "two" 3 "three"
(integer) 3
redis> ZRANGE myzset 0 1
1) "one"
2) "two"
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "one"
2) "1"
3) "two"
4) "2"
5) "three"
6) "3"
```

#### 2.3.3. ZRANGEBYLEX（已弃用）

命令语法：

```shell
ZRANGEBYLEX key min max [LIMIT offset count]
```

获取有序集合中指定字典序范围的成员。（已弃用）



#### 2.3.4. ZRANGEBYSCORE（已弃用）

命令语法：

```shell
ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT offset count]
```

获取有序集合中指定分数范围内的成员。（已弃用）



#### 2.3.5. ZRANGESTORE

命令语法：

```shell
ZRANGESTORE dest key min max [BYSCORE | BYLEX] [REV] [LIMIT offset count]
```

获取有序集合中指定范围的成员，并存储到指定的键中。（Redis 6.2+）



示例：

```shell
redis> ZADD srczset 1 "one" 2 "two" 3 "three" 4 "four"
(integer) 4
redis> ZRANGESTORE dstzset srczset 2 -1
(integer) 2
redis> ZRANGE dstzset 0 -1
1) "three"
2) "four"
redis> 
```

#### 2.3.6. ZREMRANGEBYLEX

命令语法：

```shell
ZREMRANGEBYLEX key min max
```

移除有序集合中指定字典序范围内的成员。



示例：

```shell
redis> ZADD myzset 0 aaaa 0 b 0 c 0 d 0 e
(integer) 5
redis> ZADD myzset 0 foo 0 zap 0 zip 0 ALPHA 0 alpha
(integer) 5
redis> ZRANGE myzset 0 -1
1) "ALPHA"
2) "aaaa"
3) "alpha"
4) "b"
5) "c"
6) "d"
7) "e"
8) "foo"
9) "zap"
10) "zip"
redis> ZREMRANGEBYLEX myzset [alpha [omega
(integer) 6
redis> ZRANGE myzset 0 -1
1) "ALPHA"
2) "aaaa"
3) "zap"
4) "zip"
redis> 
```

#### 2.3.7. ZREMRANGEBYRANK

命令语法：

```shell
ZREMRANGEBYRANK key start stop
```

移除有序集合中指定排名范围内的成员（按升序排序）。

start和stop都是从0开始的索引，0是分数最低的成员，-1表示分数最高的成员，以此类推。



示例：

```shell
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREMRANGEBYRANK myzset 0 1
(integer) 2
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "three"
2) "3"
redis> 
```

#### 2.3.8. ZREMRANGEBYSCORE

命令语法：

```shell
ZREMRANGEBYSCORE key min max
```

移除有序集合中指定分数范围内的成员。



示例：

```shell
redis> ZADD myzset 1 "one"
(integer) 1
redis> ZADD myzset 2 "two"
(integer) 1
redis> ZADD myzset 3 "three"
(integer) 1
redis> ZREMRANGEBYSCORE myzset -inf (2
(integer) 1
redis> ZRANGE myzset 0 -1 WITHSCORES
1) "two"
2) "2"
3) "three"
4) "3"
redis> 
```

#### 2.3.9. ZREVRANGE（已弃用）

命令语法：

```shell
ZREVRANGE key start stop [WITHSCORES]
```

获取有序集合中指定索引范围内的成员（逆序）。（已弃用）



#### 2.3.10. ZREVRANGEBYLEX（已弃用）

命令语法：

```shell
ZREVRANGEBYLEX key max min [LIMIT offset count]
```

获取有序集合中指定字典序范围内的成员（逆序）。（已弃用）




#### 2.3.11. ZREVRANGEBYSCORE（已弃用）

命令语法：

```shell
ZREVRANGEBYSCORE key max min [WITHSCORES] [LIMIT offset count]
```

获取有序集合中指定分数范围内的成员（逆序）。（已弃用）



### 2.4. 集合运算

集合运算命令用于计算多个有序集合之间的交集、并集和差集。

#### 2.4.1. ZDIFF

命令语法：

```shell
ZDIFF numkeys key [key ...] [WITHSCORES]
```

计算多个有序集合的差集。

- numkeys表示参与计算的键的数量
- WITHSCORES选项返回结果时包含分数



示例：

```shell
redis> ZADD zset1 1 "one"
(integer) 1
redis> ZADD zset1 2 "two"
(integer) 1
redis> ZADD zset1 3 "three"
(integer) 1
redis> ZADD zset2 1 "one"
(integer) 1
redis> ZADD zset2 2 "two"
(integer) 1
redis> ZDIFF 2 zset1 zset2
1) "three"
redis> ZDIFF 2 zset1 zset2 WITHSCORES
1) "three"
2) "3"
redis> 
```

#### 2.4.2. ZDIFFSTORE

命令语法：

```shell
ZDIFFSTORE destination numkeys key [key ...]
```

计算多个有序集合的差集，并将结果保存到指定的键中。

示例：

```shell
redis> ZADD zset1 1 "one"
(integer) 1
redis> ZADD zset1 2 "two"
(integer) 1
redis> ZADD zset1 3 "three"
(integer) 1
redis> ZADD zset2 1 "one"
(integer) 1
redis> ZADD zset2 2 "two"
(integer) 1
redis> ZDIFFSTORE out 2 zset1 zset2
(integer) 1
redis> ZRANGE out 0 -1 WITHSCORES
1) "three"
2) "3"
redis>
```

#### 2.4.3. ZINTER

命令语法：

```shell
ZINTER numkeys key [key ...] 
	[WEIGHTS weight [weight ...]]
  [AGGREGATE <SUM | MIN | MAX>] 
  [WITHSCORES]
```

计算多个有序集合的交集。

- WEIGHTS：为每个集合设置权重（默认为1）
- AGGREGATE：指定分数聚合方式（SUM默认，MIN取最小，MAX取最大）



示例：

```shell
redis> ZADD zset1 1 "one"
(integer) 1
redis> ZADD zset1 2 "two"
(integer) 1
redis> ZADD zset2 1 "one"
(integer) 1
redis> ZADD zset2 2 "two"
(integer) 1
redis> ZADD zset2 3 "three"
(integer) 1
redis> ZINTER 2 zset1 zset2
1) "one"
2) "two"
redis> ZINTER 2 zset1 zset2 WITHSCORES
1) "one"
2) "2"
3) "two"
4) "4"
redis>
```

#### 2.4.4. ZINTERCARD

命令语法：

```shell
ZINTERCARD numkeys key [key ...] [LIMIT limit]
```

获取多个有序集合的交集的数量。（Redis 7.0+）

- LIMIT：限制计算的最大交集数量



示例：

```shell
redis> ZADD zset1 1 "one"
(integer) 1
redis> ZADD zset1 2 "two"
(integer) 1
redis> ZADD zset2 1 "one"
(integer) 1
redis> ZADD zset2 2 "two"
(integer) 1
redis> ZADD zset2 3 "three"
(integer) 1
redis> ZINTER 2 zset1 zset2
1) "one"
2) "two"
redis> ZINTERCARD 2 zset1 zset2
(integer) 2
redis> ZINTERCARD 2 zset1 zset2 LIMIT 1
(integer) 1
redis>
```

#### 2.4.5. ZINTERSTORE

命令语法：

```shell
ZINTERSTORE destination numkeys key [key ...] 
	[WEIGHTS weight [weight ...]] 
	[AGGREGATE <SUM | MIN | MAX>]
```

计算多个有序集合的交集，并将结果保存到指定的键中。



示例：

```shell
redis> ZADD zset1 1 "one"
(integer) 1
redis> ZADD zset1 2 "two"
(integer) 1
redis> ZADD zset2 1 "one"
(integer) 1
redis> ZADD zset2 2 "two"
(integer) 1
redis> ZADD zset2 3 "three"
(integer) 1
redis> ZINTERSTORE out 2 zset1 zset2 WEIGHTS 2 3
(integer) 2
redis> ZRANGE out 0 -1 WITHSCORES
1) "one"
2) "5"
3) "two"
4) "10"
redis> 
```

#### 2.4.6. ZUNION

命令语法：

```shell
ZUNION numkeys key [key ...] 
	[WEIGHTS weight [weight ...]]
  [AGGREGATE <SUM | MIN | MAX>] 
  [WITHSCORES]
```

计算多个有序集合的并集。

示例：

```shell
redis> ZADD zset1 1 "one"
(integer) 1
redis> ZADD zset1 2 "two"
(integer) 1
redis> ZADD zset2 1 "one"
(integer) 1
redis> ZADD zset2 2 "two"
(integer) 1
redis> ZADD zset2 3 "three"
(integer) 1
redis> ZUNION 2 zset1 zset2
1) "one"
2) "three"
3) "two"
redis> ZUNION 2 zset1 zset2 WITHSCORES
1) "one"
2) "2"
3) "three"
4) "3"
5) "two"
6) "4"
redis> 
```

#### 2.4.7. ZUNIONSTORE

命令语法：

```shell
ZUNIONSTORE destination numkeys key [key ...] 
	[WEIGHTS weight [weight ...]] 
	[AGGREGATE <SUM | MIN | MAX>]
```

计算多个有序集合的并集，并将结果存储到指定的键中。



示例：

```shell
redis> ZADD zset1 1 "one"
(integer) 1
redis> ZADD zset1 2 "two"
(integer) 1
redis> ZADD zset2 1 "one"
(integer) 1
redis> ZADD zset2 2 "two"
(integer) 1
redis> ZADD zset2 3 "three"
(integer) 1
redis> ZUNIONSTORE out 2 zset1 zset2 WEIGHTS 2 3
(integer) 3
redis> ZRANGE out 0 -1 WITHSCORES
1) "one"
2) "5"
3) "three"
4) "9"
5) "two"
6) "10"
redis>
```

### 2.5. 阻塞操作

阻塞操作命令用于在有序集合中阻塞式地弹出成员。

#### 2.5.1. BZMPOP

命令语法：

```shell
BZMPOP timeout numkeys key [key ...] <MIN | MAX> [COUNT count]
```

按照分数从一个或多个有序集合中弹出一个成员。（Redis 7.0+）

- timeout：阻塞等待的超时时间（0表示永久阻塞）

示例：

```shell
## 在一个终端执行
redis> BZMPOP 0 1 zset1 MAX

## 在另一个终端执行
redis> ZADD zset1 1 "one"
(integer) 1)

## 第一个终端将返回
1) "zset1"
2) 1) "one"
   2) "1"
```

#### 2.5.2. BZPOPMAX

命令语法：

```shell
BZPOPMAX key [key ...] timeout
```

从一个或多个有序集合中弹出分数最高的成员。

- timeout：阻塞等待的超时时间（0表示永久阻塞）

示例：

```shell
## 在一个终端执行
redis> BZPOPMAX zset1 zset2 0

## 在另一个终端执行
redis> ZADD zset1 1 "one"
(integer) 1)

## 第一个终端将返回
1) "zset1"
2) "one"
3) "1"
```

#### 2.5.3. BZPOPMIN

命令语法：

```shell
BZPOPMIN key [key ...] timeout
```

从一个或多个有序集合中弹出分数最低的成员。

- timeout：阻塞等待的超时时间（0表示永久阻塞）

示例：

```shell
## 在一个终端执行
redis> BZPOPMIN zset1 zset2 0

## 在另一个终端执行
redis> ZADD zset1 1 "one"
(integer) 1)

## 第一个终端将返回
1) "zset1"
2) "one"
3) "1"
```