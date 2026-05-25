

## 1. 安装Redis

Redis支持通过多种方式安装在多种操作系统上，详情参考Redis官方：[Install Redis Open Source ](https://redis.io/docs/latest/operate/oss_and_stack/install/)



本章节演示在CentOS 7上，通过源码的形式,下载安装Redis的操作过程。

### 1.1. 下载解压安装包

1. Redis安装包的官方下载地址：[https://download.redis.io/releases/](https://download.redis.io/releases/)。开发者根据需要，选择自己要安装的版本。

2. 进入要保存安装包的目录。

   ```shell
   cd /usr/src # 我将安装包保存/usr/src目录
   ```

3. 使用wget命令，下载安装包（使用第1步获取的下载地址）。

   ```shell
   wget https://download.redis.io/releases/redis-7.4.0.tar.gz # 我这里下载最新的7.4.0版本
   ```

   命令输出：

   ```shell
   --2026-01-04 17:55:22--  https://download.redis.io/releases/redis-7.4.0.tar.gz
   正在解析主机 download.redis.io (download.redis.io)... 104.18.26.34, 104.18.27.34, 2606:4700::6812:1b22, ...
   正在连接 download.redis.io (download.redis.io)|104.18.26.34|:443... 已连接。
   已发出 HTTP 请求，正在等待回应... 200 OK
   长度：3525325 (3.4M) [application/octet-stream]
   正在保存至: “redis-7.4.0.tar.gz”
   
   100%[===============================================================>] 3,525,325   2.62MB/s 用时 1.3s
   
   2026-01-04 17:55:24 (2.62 MB/s) - 已保存 “redis-7.4.0.tar.gz” [3525325/3525325])
   ```

4. 下载完成后，检查当前目录，可以看到刚刚下载的Redis安装包

   ```shell
   ls # 查看当前目录
   ```

   输出：

   ```shell
   redis-7.4.0.tar.gz # 下载的安装包
   ```

5. 解压并查看结果

   ```shell
   tar zxvf redis-7.4.0.tar.gz # 解压
   
   ls # 查看当前目录
   ```

   ```shell
   # ls 输出
   redis-7.4.0  redis-7.4.0.tar.gz
   ```

6. 解压完成后，可以删除原安装包，当然也可以不删。

   ```shell
   rm redis-7.4.0.tar.gz
   ```

### 1.2. 构建安装

基于上面的下载解压后，就可以正式进入构建安装环节：

```shell
## 进入文件夹
cd /usr/src/redis-7.4.0

## make
make

## 安装
make install
```

安装过程中如果没有报错的话，就表示安装完成了。

前往默认安装位置`/usr/local/bin/`，可以看到redis相关的可执行文件:

```shell
cd /usr/local/bin/
ll 

## 输出
-rwxr-xr-x 1 root root  6820152 1月   4 17:53 redis-benchmark
lrwxrwxrwx 1 root root       12 1月   4 17:53 redis-check-aof -> redis-server
lrwxrwxrwx 1 root root       12 1月   4 17:53 redis-check-rdb -> redis-server
-rwxr-xr-x 1 root root  7801736 1月   4 17:53 redis-cli
lrwxrwxrwx 1 root root       12 1月   4 17:53 redis-sentinel -> redis-server
-rwxr-xr-x 1 root root 16177400 1月   4 17:53 redis-server
```



执行 make install 后，生成的可执行文件如下：

* `redis-server` ：Redis 服务器
* `redis-cli` ：Redis 命令行客户端
* `redis-benchmark` ：Redis 性能测试工具
* `redis-check-aof` ：AOF 文件检查工具
* `redis-check-rdb`：RDB 文件检查工具

### 1.3. 测试

1. 使用`redis-server` 启动Redis

   ```shell
   redis-server # 最简单的启动
   
   # 以默认配置启动Redis
   ```

   示例：

   ```shell
   [root@chinehe ~]# redis-server
   3136:C 05 Jan 2026 09:26:22.540 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
   3136:C 05 Jan 2026 09:26:22.540 * oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
   3136:C 05 Jan 2026 09:26:22.540 * Redis version=7.4.0, bits=64, commit=00000000, modified=1, pid=3136, just started
   3136:C 05 Jan 2026 09:26:22.540 # Warning: no config file specified, using the default config. In order to specify a config file use redis-server /path/to/redis.conf
   3136:M 05 Jan 2026 09:26:22.541 * monotonic clock: POSIX clock_gettime
                   _._
              _.-``__ ''-._
         _.-``    `.  `_.  ''-._           Redis Community Edition
     .-`` .-```.  ```\/    _.,_ ''-._     7.4.0 (00000000/1) 64 bit
    (    '      ,       .-`  | `,    )     Running in standalone mode
    |`-._`-...-` __...-.``-._|'` _.-'|     Port: 6379
    |    `-._   `._    /     _.-'    |     PID: 3136
     `-._    `-._  `-./  _.-'    _.-'
    |`-._`-._    `-.__.-'    _.-'_.-'|
    |    `-._`-._        _.-'_.-'    |           https://redis.io
     `-._    `-._`-.__.-'_.-'    _.-'
    |`-._`-._    `-.__.-'    _.-'_.-'|
    |    `-._`-._        _.-'_.-'    |
     `-._    `-._`-.__.-'_.-'    _.-'
         `-._    `-.__.-'    _.-'
             `-._        _.-'
                 `-.__.-'
   
   3136:M 05 Jan 2026 09:26:22.542 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
   3136:M 05 Jan 2026 09:26:22.542 * Server initialized
   3136:M 05 Jan 2026 09:26:22.542 * Ready to accept connections tcp
   ```

2. 使用`redis-cli` 客户端连接Redis

   ```shell
   redis-cli # 默认连接本机的6379端口
   ```

3. 发送测试命令

   ```shell
   ping
   ```

   示例：

   ```shell
   [root@chinehe ~]# redis-cli
   127.0.0.1:6379> ping
   PONG
   ```



如果上面的测试符合预期，则表示Redis的安装工作正式结束！！！



## 2. 配置Redis

参考官网：[Redis configuration](https://redis.io/docs/latest/operate/oss_and_stack/management/config/)

### 2.1. 配置方式

配置Redis有一下三种方式：

* **配置文件**

  配置Redis最推荐、最常用的方式。后文会详细讲解。

* **启动时使用命令行传递参数**

  你可以在启动Redis服务时，直接通过命令行传递 Redis 配置参数。

* **服务器运行时更改Redis配置**

  Redis服务器运行时，支持在不停止和重启服务的前提下实时变更Redis配置：使用`CONFIG SET`和`CONFIG GET`命令实现。

### 2.2. 配置文件

#### 2.2.1. 关于配置文件

Redis支持通过内置默认配置启动（如上文安装Redis章节中做的那样），但这种方式只推荐在测试和开发环境中使用。

配置Redis的正确做法是提供一个Redis配置文件，通常称为`redis.conf`。

> 从Redis 8开始，Redis开源版本有两个配置文件：
>
> * `redis.conf`：只包含Redis服务器的配置设置。
> * `redis-full.conf`：包含Redis服务器及所有可用组件的配置：Redis查询引擎、Redis时间序列、Redis概率数据结构。
>   * 该文件首行包含`redis.conf`，启动时可以拉取Redis服务器的配置。
>   * 该文件包含四个 `loadmodule` 指令，每个组件一个，同时加载 Redis JSON（但 JSON 没有配置参数）。
>   * 当想启用所有可用组件时，使用`redis-full.conf`。
>   * 如果你是从源码构建 Redis，并且选择在没有可用组件的情况下构建 Redis 服务器，你可以使用 `redis.conf` 作为配置文件。



每个配置文件包含若干指令，格式非常简单：

```shell
keyword argument1 argument2 ... argumentN

## keyword: 指令名称
## argument：指令参数

## 示例
replicaof 127.0.0.1 6380
```



可以使用双引号或单引号提供包含空格的字符串作为参数：

```shell
requirepass "hello world"
```

单引号字符串可以包含反斜杠转义的字符，双引号字符串还可以包含任何用反斜杠十六进制符号“\xff”编码的 ASCII 符号。



#### 2.2.2. 示例配置文件

配置指令列表、指令含义以及预期用途的注释，可以在Redis发行版附带的 `redis.conf` 和 `redis-full.conf` 文件中获得。以上文《安装Redis》章节中的安装步骤为例，解压目录中就包含`redis.conf`：

```shell
[root@chinehe redis-7.4.0]# pwd
/usr/src/redis-7.4.0 # redis安装包解压目录
[root@chinehe redis-7.4.0]# ll
总用量 288
-rw-rw-r--  1 root root   9854 7月  29 2024 00-RELEASENOTES
-rw-rw-r--  1 root root     51 7月  29 2024 BUGS
-rw-rw-r--  1 root root   5023 7月  29 2024 CODE_OF_CONDUCT.md
-rw-rw-r--  1 root root   7178 7月  29 2024 CONTRIBUTING.md
drwxrwxr-x  8 root root   4096 1月   4 17:48 deps
-rw-rw-r--  1 root root     11 7月  29 2024 INSTALL
-rw-rw-r--  1 root root  37493 7月  29 2024 LICENSE.txt
-rw-rw-r--  1 root root    151 7月  29 2024 Makefile
-rw-rw-r--  1 root root   6888 7月  29 2024 MANIFESTO
-rw-rw-r--  1 root root  23845 7月  29 2024 README.md
-rw-rw-r--  1 root root 108981 7月  29 2024 redis.conf # 配置文件
-rw-rw-r--  1 root root   1805 7月  29 2024 REDISCONTRIBUTIONS.txt
-rwxrwxr-x  1 root root    279 7月  29 2024 runtest
-rwxrwxr-x  1 root root    283 7月  29 2024 runtest-cluster
-rwxrwxr-x  1 root root   1804 7月  29 2024 runtest-moduleapi
-rwxrwxr-x  1 root root    285 7月  29 2024 runtest-sentinel
-rw-rw-r--  1 root root   1480 7月  29 2024 SECURITY.md
-rw-rw-r--  1 root root  14700 7月  29 2024 sentinel.conf
drwxrwxr-x  4 root root  12288 1月   4 17:53 src
drwxrwxr-x 11 root root   4096 7月  29 2024 tests
-rw-rw-r--  1 root root   3628 7月  29 2024 TLS.md
drwxrwxr-x  9 root root   4096 7月  29 2024 utils
```



Redis官网也可以方便的查看各个版本的配置示例文件：

- Configuration files for Redis 8.4: [redis-full.conf](https://raw.githubusercontent.com/redis/redis/8.4/redis-full.conf) and [redis.conf](https://raw.githubusercontent.com/redis/redis/8.4/redis.conf).
- Configuration files for Redis 8.2: [redis-full.conf](https://raw.githubusercontent.com/redis/redis/8.2/redis-full.conf) and [redis.conf](https://raw.githubusercontent.com/redis/redis/8.2/redis.conf).
- Configuration files for Redis 8.0: [redis-full.conf](https://raw.githubusercontent.com/redis/redis/8.0/redis-full.conf) and [redis.conf](https://raw.githubusercontent.com/redis/redis/8.0/redis.conf).
- Configuration file for Redis 7.4: [redis.conf](https://raw.githubusercontent.com/redis/redis/7.4/redis.conf).
- Configuration file for Redis 7.2: [redis.conf](https://raw.githubusercontent.com/redis/redis/7.2/redis.conf).
- Configuration file for Redis 7.0: [redis.conf](https://raw.githubusercontent.com/redis/redis/7.0/redis.conf).
- Configuration file for Redis 6.2: [redis.conf](https://raw.githubusercontent.com/redis/redis/6.2/redis.conf).



#### 2.2.3. 启用配置文件

编写好配置文件后，就可以通过在启动Redis服务时指定配置文件路径来启用配置文件了。

```shell
redis-server /path/to/redis.conf
```



以上文《安装Redis》章节安装的Redis为例，我们可以通过一下的步骤来启用配置文件：

1. 将自带的示例配置文件复制到我们要保存`redis.conf`的目录。

   ```shell
   # 创建目标目录
   mkdir /etc/redis/
   
   # 复制文件
   cp /usr/src/redis-7.4.0/redis.conf /etc/redis/
   ```

2. 修改配置文件。

   这里仅演示如何启用配置文件，就不去变更配置项了。

3. 通过配置文件启动。

   ```shell
   # 启动Redis并指定配置文件
   redis-server /etc/redis/redis.conf
   ```

### 2.3. 命令行参数配置

你可以在启动Redis服务时，直接通过命令行传递 Redis 配置参数。这对测试非常有用。以下是一个示例:

```shell
./redis-server --port 6380 --replicaof 127.0.0.1 6379
## 使用端口 6380 启动一个新的 Redis 实例，作为运行于 127.0.0.1 端口 6379 的实例的副本。
```

* 通过命令行传递的参数格式与 redis.conf 文件中完全相同，唯一不同的是关键字前缀是 `--`。

* 内部会生成内存中的临时配置文件（可能连接用户传递的配置文件），参数转换为 redis.conf 格式。

### 2.4. 运行时更改配置

Redis服务器运行时，支持在不停止和重启服务的前提下实时变更Redis配置：使用`CONFIG SET`和`CONFIG GET`命令实现。

* `CONFIG SET`：设置配置项
  * 所有使用 `CONFIG SET` 设置的配置参数都会被 Redis 立即加载，并从下一个执行命令开始生效。
  * 所有支持的参数都与` redis.conf` 文件中使用的配置参数含义相同。
* ``CONFIG GET`：获取配置项。
* `CONFIG REWRITE`：重写服务器启动时使用的 `redis.conf` 文件，并对其进行最小的修改，使其反映服务器当前使用的配置。
  * 自动扫描配置文件并更新与当前配置值不匹配的字段。设置为默认值的字段不会被添加。配置文件中的注释会被保留。
  * 用于动态修改配置项后，同步修改配置文件。

**特别注意：**

* 并非所有的配置指令都支持这种方式，但大多数多支持。通过发送 `CONFIG GET *` 命令，可以获得 `CONFIG SET` 支持的配置参数列表。
*  `CONFIG SET` 动态修改的配置不会影响`redis.conf` 和 `redis-full.conf` 文件。所以在下一次重启 Redis 时，文件中的旧配置会被使用。因此正常情况下，都需要根据动态修改的配置，同步修改配置文件。可以手动修改配置文件，也可以使用`CONFIG REWRITE`命令。



示例：

1. 以配置文件启动Redis

   ```shell
   redis-server /etc/redis/redis.conf # 该配置文件未做修改，所有配置项均为默认配置
   ```

2. 新开终端，连接Redis并测试命令

   ```shell
   [root@chinehe ~]# redis-cli 							# 连接Redis
   127.0.0.1:6379> config get appendonly 		# 获取 appendonly 配置
   1) "appendonly"
   2) "no"																		# 默认配置为no
   127.0.0.1:6379> config set appendonly yes # 修改配置为yes
   OK
   127.0.0.1:6379> config get appendonly			# 再次获取 appendonly 配置
   1) "appendonly"
   2) "yes"																	# 修改后配置为yes
   127.0.0.1:6379> config rewrite						# 重写配置文件
   OK
   127.0.0.1:6379>
   ```

3. 检查配置文件

   ```shell
   [root@chinehe ~]# cat /etc/redis/redis.conf | grep appendonly
   
   appendonly yes   # 配置文件已经自动更新
   # For example, if appendfilename is set to appendonly.aof, the following file
   # - appendonly.aof.1.base.rdb as a base file.
   # - appendonly.aof.1.incr.aof, appendonly.aof.2.incr.aof as incremental files.
   # - appendonly.aof.manifest as a manifest file.
   appendfilename "appendonly.aof"
   appenddirname "appendonlydir"
   ```

   

## 3. 连接Redis

在上文中，我们都是使用Redis自带的`redis-cli`客户端工具来连接Redis服务。

除此之外，连接Redis服务的方式还有很多，常见的方式有：

* `redis-cli`：命令行工具。
* `Redis Insight` ：图形界面工具。
* IDE插件：常用的IDE如VS Code、IDEA、Goland等都有Redis插件。
* Redis client API：在代码中使用Redis客户端哭连接Redis。



## 4. Redis命令

客户端应用程序或工具**通过命令与Redis进行交互**。大多数命令用于实现各种数据类型数据的存储和检索，但也有部分命令用于服务器配置、安全等。



Redis命令的完整列表可以参考官网： [Redis commands reference](https://redis.io/docs/latest/commands/) 