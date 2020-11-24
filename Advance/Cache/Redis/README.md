# 简介

一个最简单的键值数据库需要包括访问框架、索引模块、操作模块、存储模块。

- 访问模式通常有两种：
  - 通过函数库调用的方式供外部应用使用；
  - 通过网络框架以 Socket 通信的形式对外提供键值对操作

Redis(REmote Dictionary Server) 远程字典服务器。开源免费，C语言编写的高性能的KV键值分布式内存数据库，基于内存运行，并支持持久化的 NoSQL 数据库。

- 速度快：数据保存在内存、底层使用C语言、单线程；
- Redis支持数据的持久化，可将内存中的数据保持在磁盘中，重启的时候再次加载进行使用；
- 支持多种客户端语言；
- 功能丰富：发布订阅、支持Lua脚本、简单的事务、pipeline；
- Redis支持数据的备份，即master-slave模式的数据备份；
- 高可用、分布式。

Redis命令的生命周期：

Client发送命令 ——> Redis对所有命令排队 ——> Redis逐个执行命令 ——> Redis向客户端返回结果。



# 应用场景

典型应用场景：缓存系统、计数器、消息队列系统、排行榜、社交网络、实时系统。

- 配合关系型数据库做高速缓存

  - 高频次，热门访问的数据，降低数据库IO
  - 分布式架构，做session共享

- 由于其拥有持久化能力,利用其多样的数据结构存储特定的数据。

  | 数据类型                         | 使用                                            |
  | -------------------------------- | ----------------------------------------------- |
  | 通过List实现按自然时间排序的数据 | 最新N个数据(实现最新消息的排行)                 |
  | 利用zset(有序集合)               | (以某个条件为权重排序)排行榜，Top N，           |
  | Expire过期                       | 时效性的数据，比如手机验证码                    |
  | 原子性，自增方法INCR、DECR       | 计数器，秒杀                                    |
  | 利用Set集合(可自动排重)          | 去除大量数据中的重复数据                        |
  | 利用list集合                     | 模拟消息队列(push命令存储任务，pop命令取出任务) |
  | pub/sub模式                      | 发布订阅消息系统                                |
  | Hash                             | 存储用户信息。                                  |



# 安装

下载见[官网](https://redis.io/)

Linux下安装：

1. `wget http://download.redis.io/releases/redis-3.0.7.tar.gz`
2. `tar -xzf redis-3.0.7.tar.gz`
3. `ln -s redis-5.0.3 redis`建立一个软连接，便于以后升级
4. `cd redis`（进入文件夹）
5. 执行`make`
   - 可能出现错误：gcc 命令未找到。需要安装gcc
     - 联网：`yum install gcc`。安装后再次`make`
   - 提示Jemalloc/jemalloc.h:没有那个文件或目录
     - 运行`make distclean`后再`make`
6. 执行`make install`

---

安装后的目录：

- 查看默认安装目录：/usr/local/bin
  - `redis-benchmark`：性能测试工具(服务启动以后执行)
  - `redis-check-aof`：修复有问题的AOF文件
  - `redis-check-dump`：修复有问题的dump.rdb文件
  - `redis-sentinel`：Redis集群使用
  - `redis-server`Redis服务器启动命令
  - `redis-cli`：客户端，操作入口



# 启动

` cat redis.conf |grep -v "#" | grep -v "^$"`查看Redis的配置文件（不显示注释、空格）。

1. 备份redis.conf：
   - 在Redis的解压目录下`mkdir config`，将解压目录中的redis.conf复制一份到该目录下：`cp redis.conf ./config/redis.conf`
2. 如果想要Redis作为后台服务启动，编辑redis.conf文件，修改`daemonize no`为`yes`
3. Redis单实例服务端启动：在安装目录(可以在任意目录下)下`redis-server`，
   - 直接使用该命令会占用整个窗口(无法再进行其他操作)，命令后加上` &`，在启动Redis服务端后继续使用命令行；
   - 指定端口：`--port 端口`；
   - 也可以使用指定的配置文件启动：`redis-server redis.conf`，通常都是使用配置文件启动
4. Redis单实例客户端启动：`redis-cli`
   - 指定端口启动`redis-cli -p 6380`。6379是默认端口。然后提示符会发生改变
   - 不使用`-p`的话，默认连接6379端口
   - 输入密码：`-a 密码`
   - 连接远程的Redis服务端：`-h 远程地址`
   - 查看Redis有没有启动：`ps -ef|grep redis`
5. 关闭服务：`redis-cli shutdown`，会触发持久化
   - 也可以进入客户端后停止Redis：输入`SHUTDOWN`，然后`quit`
   - 指定端口关闭：`redis-cli -p 6379 shutdown`



访问密码的查看、设置和取消。默认是不设密码的

可以在终端进入Redis后输入：

- `config get requirepass`：查询密码
- `config set requirepass 密码`：设置密码
- `auth 密码`：输入密码



# 过期策略

Redis 是如何对过期的数据进行删除的呢？

- 定期删除：Redis 默认是每隔100ms就随机抽取**一些**设置了过期时间的key，检查其是否过期，如果过期就删除。
  - 定期删除可能会导致很多过期key到了时间并没有被删除掉，怎么办？使用惰性删除
- 惰性删除：你获取某个key的时候，Redis会检查一下，这个key如果设置了过期时间那么是否过期了？如果过期了此时就会删除，不会给你返回任何东西。

通过上述两种手段结合起来，保证过期的key一定会被干掉。但是这仍然有问题，如果定期删除漏掉了很多过期key，然后你也没及时去查，也就没走惰性删除，此时会怎么样？如果大量过期key堆积在内存里，导致Redis内存块耗尽了，怎么办？内存淘汰机制。

如果Redis的内存占用过多的时候，此时会进行内存淘汰，有如下一些策略：

- noeviction：当内存不足以容纳新写入数据时，新写入操作会报错
- allkeys-lru：当内存不足以容纳新写入数据时，在键空间中，移除最近最少使用的key。（常用）
- allkeys-random：当内存不足以容纳新写入数据时，在键空间中，随机移除某个key
- volatile-lru：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，移除最近最少使用的key
- volatile-random：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，随机移除某个key
- volatile-ttl：当内存不足以容纳新写入数据时，在设置了过期时间的键空间中，有更早过期时间的key优先移除



# 了解

Redis默认16个数据库，类似数组下标从0开始，初始默认使用0号库。`select`命令切换数据库，如：`select 7`。

统一密码管理，16个库都是同样密码，要么都OK要么一个也连接不上。

Redis 的索引都是从零开始的。

Redis 是单线程+多路IO复用技术：

- Redis 是**单进程**来处理客户端的请求。对读写等事件的响应是通过 epoll 函数的包装做到的，Redis 的实际处理速度完全依靠主进程的执行效率。
  - 单进程保证了 Redis 单命令的原子性
    - 原子操作是指不会被线程调度机制打断的操作；这种操作一旦开始，就一直运行到结束，中间不会有任何context switch （切换到另一个线程）；
    - 在单线程中， 能够在单条指令中完成的操作都可以认为是"原子操作"，因为中断只能发生于指令之间；
    - 在多线程中，不能被其它进程（线程）打断的操作就叫原子操作
- 多路复用是指使用一个线程来检查多个文件描述符（Socket）的就绪状态，比如调用select和poll函数，传入多个文件描述符，如果有一个文件描述符就绪，则返回，否则阻塞直到超时。得到就绪状态后进行真正的操作可以在同一个线程里执行，也可以启动线程执行（比如使用线程池）。

Q：一般而言，使用单线程是非常慢的，那Redis使用单线程为什么会很快？

A：**纯内存；非阻塞IO；单线程避免了线程切换和竞态消耗**。



一次只执行一条命令，拒绝长(慢)命令。

---

出现错误：

> (error) MISCONF Redis is configured to save RDB snapshots, but it is currently not able to persist on disk. Commands that may modify the data set are disabled, because this instance is configured to report errors during writes if RDB snapshotting fails (stop-writes-on-bgsave-error option). Please check the Redis logs for details about the RDB error.

Redis被配置为保存数据库快照，但它目前不能持久化到硬盘。用来修改集合数据的命令不能用。请查看Redis日志的详细错误信息。

在Redis的命令行输入：`config set stop-writes-on-bgsave-error no`



# Redis压测

- `redis-benchmark -h 127.0.0.1 -p 6379 -c 100 -n 100000`：100个并发连接，100000个请求
- `redis-benchmark -h 127.0.0.1 -p 6379 -q -d 100`：存取大小为100B的数据包
  - `-q`：只做一些简单的输出
- `redis-benchmark -t set,lpush -q -n 100000`：只测试`set`、`lpush`这两个操作在100000个请求时的性能
- `redis-benchmark -n 10000 -q script load "redis.call('set','foo','bar')"`只测试`redis.call('set','foo','bar')`这一条命令在100000个请求时的性能



# 常见配置

- Redis默认不是以守护进程的方式运行，可通过配置`daemonize no`为`yes`来修改；

- 当Redis以守护进程方式运行时，Redis默认会把 pid 写入 /var/run/redis.pid 文件，可通过pidfile指定

  - `pidfile /var/run/redis.pid`

- 指定Redis监听端口，默认为6379：`port 6379`；

- 绑定主机地址：`bind 127.0.0.1`，远程访问时须注释这条配置；

- `timeout 300`：当客户端闲置多长时间后关闭连接，如果指定为0，表示关闭该功能；

- `loglevel notice`指定日志记录级别，Redis支持四种级别：debug、verbose、notice、warning，默认notice；

- `logfile ""`日志记录方式，默认为标准输出，如果配置Redis为守护进程运行，且这里又配置为日志记录输出为标准输出，则日志将会发送给 /dev/null

- `databases 16`设置数据库数量；

- `save <seconds> <changes>`指定多长时间内又多少次更新操作就将数据同步到数据文件，可以多个条件配合

  ```shell
  Redis默认配置文件中提供了三个条件：
  save 900 1    #900秒内有一个更改
  save 300 10    #300秒内有10个更改
  save 60 10000  #60秒内有10000个更改
  ```

- 指定存储至本地数据库时是否压缩数据，默认yes，Redis采用LZF压缩，如果为了节省CPU时间，可以关闭该选项，但会导致数据库文件变得巨大：`rdbcompression yes`；

- `dbfilename dump.rdb`指定本地数据库文件名，默认为 dump.rdb；

- `dir ./`指定本地数据库存放目录；

- `slaveof <masterip> <masterport>`设置当本机为 slav 服务时，设置master服务的IP地址及端口，在Redis启动时，会自动从master进行数据同步；

- 当master服务设置了密码保护时，slav服务连接master的密码：`masterauth <master-password>`；

- `requirepass foobared`设置Redis连接密码，如果配置了连接密码，客户端在连接Redis时需要通过 `AUTH <password>`命令提供密码，默认关闭；

- `maxclients 128`设置同一时间最大客户端连接数，默认无限制，Redis可同时打开的客户端连接数为Redis进程可以打开的最大文件描述符数，如果设置`maxclients 0`表示不做限制。当客户端连接数达到限制时，Redis会关闭新的连接，并向客户端返回 max number of clients reached 错误信息；

- `maxmemory <bytes>`指定Redis最大内存限制，Redis在启动时会把数据加载到内存中，达到最大内存后，Redis会先尝试清除已到期或即将到器的key，当此方法处理后，仍然达最大内存设置，将无法再进行写入操作，但仍可进行读取操作。Redis新的VM机制会把key存放内存、value存放在swap区；

- `appendonly no`指定是否再每次更新会进行日志记录，Redis默认把数据写入磁盘，如果不写入，可能会在断电时导致一段时间内的数据丢失。因为Redis本身同步数据文件是按照上面save条件来同步的，所以有的数据会在一段时间内只存在于内存中。默认为no；

- 指定更新日志文件名，默认：`appendfilename appendonly.aof`；

- `appendfsync everysec`指定更新日志条件，共3个可选值：

  - no：表示等早做系统进行数据缓存同步到磁盘(快)
  - always：表示每次更新操作后手动调用 fsync() 将数据写到磁盘(慢、安全)
  - everysec：表示美妙同步一次(默认值)

- ` vm-enabled no`指定是否启用虚拟内存机制，默认值为no，简单的介绍一下，VM机制将数据分页存放，由Redis将访问量较少的页即冷数据swap到磁盘上，访问多的页面由磁盘自动换出到内存中（在后面的文章我会仔细分析Redis的VM机制）

- ` vm-swap-file /tmp/redis.swap`虚拟内存文件路径，默认值为/tmp/redis.swap，不可多个Redis实例共享

- `vm-max-memory 0`将所有大于vm-max-memory的数据存入虚拟内存,无论vm-max-memory设置多小,所有索引数据都是内存存储的(Redis的索引数据 就是keys),也就是说,当vm-max-memory设置为0的时候,其实是所有value都存在于磁盘。默认值为0

- `vm-page-size 32`，Redis swap文件分成了很多的page，一个对象可以保存在多个page上面，但一个page上不能被多个对象共享，vm-page-size是要根据存储的 数据大小来设定的，作者建议如果存储很多小对象，page大小最好设置为32或者64bytes；如果存储很大大对象，则可以使用更大的page，如果不 确定，就使用默认值

- `vm-pages 134217728`设置swap文件中的page数量，由于页表（一种表示页面空闲或使用的bitmap）是在放在内存中的，，在磁盘上每8个pages将消耗1byte的内存。

- `vm-max-threads 4`设置访问swap文件的线程数,最好不要超过机器的核数,如果设置为0,那么所有对swap文件的操作都是串行的，可能会造成比较长时间的延迟。默认值为4

- `glueoutputbuf yes`设置在向客户端应答时，是否把较小的包合并为一个包发送，默认为开启

- 指定在超过一定的数量或者最大的元素超过某一临界值时，采用一种特殊的哈希算法

  ```shell
  hash-max-zipmap-entries 64
  hash-max-zipmap-value 512
  ```

- `activerehashing yes`指定是否激活重置哈希，默认为开启

- `include /path/to/local.conf`指定包含其它的配置文件，可以在同一主机上多个Redis实例之间使用同一份配置文件，而同时各个实例又拥有自己的特定配置文件


