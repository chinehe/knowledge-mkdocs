Redis是一个功能强大、性能卓越的内存数据存储系统，它不仅可以用作缓存，还能用于会话存储、消息队列、排行榜等多种场景。其丰富的数据类型、灵活的持久化机制和高可用性特性，使其成为现代Web应用架构中不可或缺的组件。

## 1. Redis简介

### 1.1. Redis是什么？

Redis（Remote Dictionary Server，远程字典服务器）是一个开源的非关系型数据库，Redis 可以用作数据库、缓存、流引擎、消息代理等。
Redis将数据存储在内存中，以获得极高的读写性能，同时提供持久化功能将内存中的数据保存到磁盘上，以防止服务器重启时数据丢失。

> 可以把Redis想象成一个功能强大的内存缓存系统，它不仅能够快速读写数据，还支持丰富的数据结构和操作，是现代Web应用中不可或缺的组件。

Redis相关资源：

- [Redis官网](https://redis.io/)
- [Redis官方文档](https://redis.io/docs/latest/)
- [Redis源码地址](https://github.com/redis/redis)

### 1.2. Redis的起源

2008年萨尔瓦多在开发一个进行网站实时统计软件项目（LLOOGG），项目的实时统计功能需要频繁的进行数据库的读写(对数据库的读写要求很高—数千次/s)，MySQL满足不了项目的需求，萨尔瓦多就使用C语言自定义了一个数据存储系统—Redis。考虑到最终限制数据库性能的瓶颈在于磁盘，他自己去实现一个具有列表结构的数据库的原型，把数据放在内存而不是磁盘，这样可以大大地提升列表的 push 和 pop 的效率。萨尔瓦多发现这种思路确实能解决这个问题，所以用 C 语言重写了这个内存数据库，并且加上了持久化的功能。2009 年，Redis 横空出世了。后来萨尔瓦多不满足仅仅在LLOOGG这个项目中使用redis，就对redis进行产品化并进行开源，以便让更多的⼈能够使用。

> 来源：[Redis起源](https://cloud.tencent.com/developer/article/2440766)

从最开始只支持列表的数据库，到现在支持多种数据类型，并且提供了一系列的高级特性，Redis 已经成为一个在全世界被广泛使用的开源项目。

### 1.3. 为什么选择Redis？

> 摘自github：[why choose redis](https://github.com/redis/redis?tab=readme-ov-file#why-choose-redis)

Redis is a popular choice for developers worldwide due to its combination of speed, flexibility, and rich feature set. Here's why people choose Redis for:
Redis 因其速度、灵活性和丰富的功能集而受到全球开发者的喜爱。以下是人们选择 Redis 的原因：

- **Performance:** Because Redis keeps data primarily in memory and uses efficient data structures, it achieves extremely low latency (often sub-millisecond) for both read and write operations. This makes it ideal for applications demanding real-time responsiveness.
  **性能：** 由于 Redis 主要将数据存储在内存中并采用高效的数据结构，因此在读写作中实现了极低的延迟（通常为亚毫秒）。这使得它非常适合需要实时响应的应用。

- **Flexibility:** Redis isn't just a key-value store, it provides native support for a wide range of data structures and capabilities listed in [What is Redis?](https://github.com/redis/redis?tab=readme-ov-file#what-is-redis)
  **灵活性：**Redis 不仅仅是一个键值存储，它还原生支持《[ 什么是 Redis？](https://github.com/redis/redis?tab=readme-ov-file#what-is-redis)》中列出的各种数据结构和功能。

- **Extensibility:** Redis is not limited to the built-in data structures, it has a [modules API](https://redis.io/docs/latest/develop/reference/modules/) that makes it possible to extend Redis functionality and rapidly implement new Redis commands
  **可扩展性：**Redis 不仅限于内置数据结构，它还有一个[模块 API](https://redis.io/docs/latest/develop/reference/modules/)，可以扩展 Redis 功能并快速实现新的 Redis 命令。

- **Simplicity:** Redis has a simple, text-based protocol and [well-documented command set](https://redis.io/docs/latest/commands/)
  **简洁：**Redis 拥有简单的文本协议和[文档完善的命令集](https://redis.io/docs/latest/commands/)。

- **Ubiquity:** Redis is battle tested in production workloads at a massive scale. There is a good chance you indirectly interact with Redis several times daily
  **普及性：**Redis 在大规模生产工作负载中经过了实战考验。你很可能每天多次间接与 Redis 互动。

- **Versatility**: Redis is the de facto standard for use cases such as:
  **多样性** ：Redis 是以下用例的事实标准：

  - **Caching:** quickly access frequently used data without needing to query your primary database
    **缓存：** 无需查询主数据库即可快速访问常用数据。
  - **Session management:** read and write user session data without hurting user experience or slowing down every API call
    **会话管理：** 读写用户会话数据，同时不损害用户体验或减缓每次 API 调用。
  - **Querying, sorting, and analytics:** perform deduplication, full text search, and secondary indexing on in-memory data as fast as possible
    **查询、排序和分析：** 尽可能快速地对内存数据进行重复删除、全文搜索和二次索引。
  - **Messaging and interservice communication:** job queues, message brokering, pub/sub, and streams for communicating between services
    **消息传递与服务间通信：** 作业队列、消息代理、发布/订阅以及服务间通信流。
  - **Vector operations:** Long-term and short-term LLM memory, RAG content retrieval, semantic caching, semantic routing, and vector similarity search
    **向量运算：** 长期和短期 LLM 记忆、RAG 内容检索、语义缓存、语义路由和向量相似性搜索。

In summary, Redis provides a powerful, fast, and flexible toolkit for solving a wide variety of data management challenges. 
总之，Redis 提供了一套强大、快速且灵活的工具包，用于解决各种数据管理难题。

## 2. Redis的核心特性

### 2.1. 高性能

Redis的最大特点是其卓越的性能。由于数据存储在内存中，Redis的读写速度非常快，通常情况下每秒可以处理10万次以上的读写操作。根据官方测试数据，Redis在普通硬件上可以达到每秒约110,000次SET操作和每秒约81,000次GET操作。

这种高性能得益于以下因素：
- 数据存储在内存中，访问速度极快
- 单线程事件循环模型，避免了线程切换的开销
- 高效的数据结构设计
- 非阻塞I/O操作

> 传统 Redis：核心网络 I/O 和命令处理是单线程的
> Redis 6.0+：引入了多线程来处理网络 I/O，但命令执行仍是单线程

### 2.2. 丰富的数据类型

Redis不仅仅是一个简单的键值存储系统，它支持多种数据结构，这使得它比传统的键值存储更加灵活和强大：

1. **字符串（Strings）**：最基本的类型，可以存储文本、数字或二进制数据（如图片）。
2. **哈希（Hashes）**：用于存储字段和值的映射，适合存储对象。
3. **列表（Lists）**：按插入顺序排序的字符串元素集合，可以从两端进行插入和删除操作。
4. **集合（Sets）**：无序的字符串集合，不允许重复元素，支持集合间的交、并、差运算。
5. **有序集合（Sorted Sets）**：类似于集合，但每个元素都关联一个分数，元素按分数排序。
6. **位图（Bitmaps）**：基于字符串的位操作，可以高效地处理大量布尔值。
7. **超日志（HyperLogLogs）**：用于估计集合中唯一元素的数量。
8. **地理空间（Geospatial）**：存储地理位置信息并支持位置查询。

这些也只是Redis支持的数据类型的一部分，另外还有Vector Set向量集、Stream流、JSON等等。可以参考官网：[Redis Data Type](https://redis.io/docs/latest/develop/data-types/)

Redis 不仅限于内置数据结构，它还有一个[模块 API](https://redis.io/docs/latest/develop/reference/modules/)，可以扩展 Redis 功能并快速实现新的 Redis 命令。

### 2.3. 持久化机制

虽然Redis是内存数据库，但它提供了多种持久化机制来确保数据的安全性：

1. **RDB（Redis Database）**：在指定的时间间隔内将内存中的数据集快照写入磁盘。这种机制适合备份，恢复速度较快，但可能丢失最后一次快照后的数据。

2. **AOF（Append Only File）**：记录每个写操作命令，当Redis重启时会重新执行这些命令来恢复数据。AOF提供了更好的数据安全性，但文件通常比RDB大，恢复速度较慢。

3. **混合持久化**：Redis 4.0引入的新机制，结合了RDB和AOF的优点，在AOF中使用RDB快照和追加写命令。

### 2.4. 复制与高可用

Redis支持主从复制（Master-Slave Replication），可以将一个Redis服务器的数据复制到多个从服务器。这种机制提供了以下好处：

- 数据冗余：实现数据的热备份
- 读写分离：主服务器处理写操作，从服务器处理读操作
- 高可用：当主服务器宕机时，可以快速切换到从服务器

此外，Redis还提供了Sentinel（哨兵）系统，用于监控Redis主从服务器的运行状态，并在主服务器宕机时自动进行故障转移。

### 2.5. 分片

Redis支持分片（Sharding）机制，可以将数据分布在多个Redis实例上，以提高系统的存储能力和处理能力。Redis Cluster是官方提供的分布式解决方案，支持自动分片和高可用性。

### 2.6. 丰富的功能

Redis还提供了一些高级功能，比如：

1. **事务**：支持简单的事务功能，可以将多个命令打包执行。
2. **发布/订阅**：提供发布/订阅模式，可以用于实现消息队列等功能。
3. **Lua脚本**：支持使用Lua脚本执行复杂的操作，减少网络往返次数。
4. **键过期**：支持为键设置过期时间，Redis会自动删除过期的键。
5. **流水线**：支持将多个命令打包发送，减少网络延迟的影响。

## 3. Redis的应用场景

下面是Redis的常用场景：

> 当然，Redis的应用场景远不止这些！

* **缓存系统**

  Redis最常用的应用场景是作为缓存系统。由于其高性能的读写能力，Redis可以显著提高Web应用的响应速度。通常将热点数据存储在Redis中，减少对后端数据库的访问压力。

* **会话存储**

  在分布式系统中，Redis常用于存储用户会话信息。多个应用服务器可以共享存储在Redis中的会话数据，实现用户状态的统一管理。

* **消息队列**

  Redis的列表和发布/订阅功能使其成为一个轻量级的消息队列系统，可以用于处理异步任务、实现应用间的解耦。

* **排行榜/计数器**

  Redis的有序集合特性使其非常适合实现排行榜功能。例如，可以使用ZADD命令将用户的分数添加到有序集合中，然后使用ZRANGE命令获取排名。

* **实时分析**

  Redis的高性能和丰富的数据结构使其适合用于实时数据分析，如统计网站访问次数、在线用户数等。

  

上面这些场景比较实际，**Redis官网从更高层次的角度列举了一些Redis的主要使用场景**：

* **Caching:** Supports multiple eviction policies, key expiration, and hash-field expiration.
  **缓存：** 支持多种驱逐策略、密钥过期和哈希字段过期。
* **Distributed Session Store:** Offers flexible session data modeling (string, JSON, hash).
  **分布式会话存储：** 提供灵活的会话数据建模（字符串、JSON、哈希）。
* **Data Structure Server:** Provides low-level data structures (strings, lists, sets, hashes, sorted sets, JSON, etc.) with high-level semantics (counters, queues, leaderboards, rate limiters) and supports transactions & scripting.
  **数据结构服务器：** 提供低层数据结构（字符串、列表、集合、哈希、排序集、JSON 等），并支持高级语义（计数器、队列、排行榜、速率限制器），并支持事务和脚本。
* **NoSQL Data Store:** Key-value, document, and time series data storage.
  **NoSQL 数据存储：** 键值、文档和时间序列数据存储。
* **Search and Query Engine:** Indexing for hash/JSON documents, supporting vector search, full-text search, geospatial queries, ranking, and aggregations via Redis Query Engine.
  **搜索与查询引擎：** 为哈希/JSON 文档提供索引，支持矢量搜索、全文搜索、地理空间查询、排名和通过 Redis 查询引擎进行聚合。
* **Event Store & Message Broker:** Implements queues (lists), priority queues (sorted sets), event deduplication (sets), streams, and pub/sub with probabilistic stream processing capabilities.
  **活动商店与消息代理：** 实现队列（列表）、优先队列（排序集合）、事件去重（集合）、流以及具有概率流处理能力的发布/订阅。
* **Vector Store for GenAI:** Integrates with AI applications (e.g. LangGraph, mem0) for short-term memory, long-term memory, LLM response caching (semantic caching), and retrieval augmented generation (RAG).
  **生成式人工智能的向量存储器：** 与人工智能应用（如 LangGraph、mem0）集成，用于短期记忆、长期记忆、LLM 响应缓存（语义缓存）和检索增强生成（RAG）。
* **Real-Time Analytics:** Powers personalization, recommendations, fraud detection, and risk assessment.
  **实时分析：** 具备个性化、推荐、欺诈检测和风险评估的能力。

## 4. Redis与其他数据库的比较

### 4.1. 与关系型数据库的比较

| 特性 | Redis | 关系型数据库 |
|------|-------|-------------|
| 数据模型 | 键值对，多种数据结构 | 表格结构，行和列 |
| 性能 | 极高（内存存储） | 中等（磁盘存储） |
| 持久性 | 可选（RDB、AOF） | 默认持久化 |
| 查询语言 | 简单命令 | SQL |
| ACID支持 | 有限支持 | 完整支持 |
| 适用场景 | 缓存、会话存储 | 复杂查询、事务处理 |

### 4.2. 与Memcached的比较

| 特性 | Redis | Memcached |
|------|-------|-----------|
| 数据类型 | 丰富的数据类型 | 简单的键值对 |
| 持久化 | 支持 | 不支持 |
| 多线程 | 单线程 | 多线程 |
| 内存效率 | 较高 | 非常高 |
| 功能丰富度 | 丰富 | 相对简单 |

> 传统 Redis：核心网络 I/O 和命令处理是单线程的
> Redis 6.0+：引入了多线程来处理网络 I/O，但命令执行仍是单线程

## 5. Redis版本历史

Redis自2009年首次发布以来，经历了多个重要版本的迭代，每个版本都引入了新的功能和性能改进。以下是Redis发展过程中的一些重要版本及其变更：

### 5.1 Redis 1.0-2.0（2009-2010年）

Redis最初的几个版本奠定了其基础架构。2009年5月，第一个稳定版本发布，支持基本的字符串、列表、集合等数据类型。这个阶段Redis主要是一个简单的内存键值存储系统。

### 5.2 Redis 2.2（2011年9月）

这个版本引入了两个重要的功能：
- **Pub/Sub（发布/订阅）**：允许客户端订阅频道并接收发布到这些频道的消息
- **配置动态重载**：支持在不重启服务的情况下重新加载配置

### 5.3 Redis 2.4（2011年12月）

Redis 2.4版本主要关注稳定性和性能优化，同时改进了内存管理和持久化机制。

### 5.4 Redis 2.6（2012年12月）

这是Redis发展史上的一个重要里程碑，引入了：
- **Lua脚本支持**：允许在服务器端执行Lua脚本，大大增强了Redis的编程能力
- **Bit Operations（位操作）**：对字符串类型增加了位操作命令，如GETBIT、SETBIT等
- **Object Commands（对象命令）**：新增了如OBJECT命令，用于检查内存使用情况和内部状态

### 5.5 Redis 2.8（2013年11月）

这个版本引入了几个关键功能：
- **Redis Sentinel（哨兵）**：提供了高可用性解决方案，可以监控、通知、自动故障转移
- **Redis Replication优化**：改进了主从复制机制，增加了部分重同步功能，减少网络中断时的全量同步
- **集群支持预览**：虽然未完全完成，但为后续的集群功能奠定了基础

### 5.6 Redis 3.0（2015年4月）

这是Redis历史上最重要的版本之一，因为：
- **Redis Cluster（集群）**：正式发布了Redis集群功能，提供了自动分片和高可用性
- 集群功能使得Redis可以水平扩展，支持更大的数据量和更高的并发访问

### 5.7 Redis 3.2（2016年5月）

这个版本增加了：
- **Geo Commands（地理空间命令）**：新增了地理位置相关的命令，如GEOADD、GEORADIUS等
- **Redis on Windows**：微软发布了Redis在Windows上的正式支持版本
- **ACLs（访问控制列表）**：引入了更细粒度的访问控制机制

### 5.8 Redis 4.0（2017年7月）

Redis 4.0带来了重大改进：
- **LFU/LRU improvements（缓存淘汰策略改进）**：提供了更精确的缓存淘汰算法
- **Redis Modules**：允许开发者创建Redis模块，扩展Redis的功能
- **Active Memory Defragmentation（主动内存碎片整理）**：减少了内存碎片，提高了内存使用效率
- **混合持久化**：AOF重写时使用RDB格式，结合了RDB和AOF的优点

### 5.9 Redis 5.0（2018年7月）

Redis 5.0的主要特性是：
- **Streams（流数据类型）**：引入了全新的Stream数据类型，支持消息队列功能，具有持久化和消费组等特性
- Stream数据类型使得Redis成为一个功能强大的消息中间件

### 5.10 Redis 6.0（2020年5月）

这是Redis的重大更新，主要特性包括：
- **ACL（访问控制列表）**：正式支持细粒度的用户权限管理
- **Client Side Caching（客户端缓存）**：通过RESP3协议支持客户端缓存，减少不必要的网络请求
- **Threaded I/O（线程I/O）**：引入了多线程I/O，显著提高了网络处理性能
- **Redis Cluster Proxy**：为集群提供了代理支持

### 5.11 Redis 7.0（2022年4月）

Redis 7.0是最新的主要版本，引入了：
- **Functions（函数）**：新的脚本功能，替代了传统的EVAL命令，提供了更好的性能和安全性
- **Command Tips**：为命令提供了更详细的使用提示
- **ACL日志**：记录ACL相关的操作日志
- **性能改进**：在多个方面进行了性能优化