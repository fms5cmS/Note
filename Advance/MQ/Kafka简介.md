
Apache Kafka 是消息引擎系统，也是一个分布式流处理平台（Distributed Streaming Platform）！

Kafka 的灵魂 [The Log: What every software engineer should know about real-time data's unifying abstraction](https://engineering.linkedin.com/distributed-systems/log-what-every-software-engineer-should-know-about-real-time-datas-unifying)

Kafka 会将结构化 的消息（Record）转换成**二进制的字节序列**来进行传输。

Kafka 的消息模型是发布-订阅模型，其主要设计目标：

- 以时间复杂度为 O(1) 的方式提供消息持久化能力，即使对 TB 级以上数据也能保证常数时间复杂度的访问性能；
- 高吞吐率。即使在非常廉价的商用机器上也能做到单机支持每秒 100K 条以上消息的传输；
- 支持 Kafka Server 间的消息分区，及分布式消费，同时保证每个 Partition 内的消息顺序传输；
- 同时支持实时数据处理和离线数据处理（从一个很早的 offset 开始回放等）；
- Scale out：支持在线水平扩展；

> Kafka 使用消息日志来保存数据，且只能追加写，避免了缓慢的随机 I/O 操作，改为性能较好的顺序 I/O 写操作

# 核心概念

Kafka 集群包含一个或多个服务器，每个服务器节点称为一个 Broker。Broker 存储 Topic 的数据。

Topic 是逻辑概念，真正在 Broker 间分布的是 Partition！每条消息被发送到 Broker，会根据 Partition 规则选择存储到哪个 Partition。所以 Partition 规则需要尽量保证消息可以均匀分布到不同 Partition。

## 分区机制（Partitioning）

Kafka 的伸缩性（Scalability）问题是通过分区机制来解决的。

Kafka 的分区机制会**将每个 Topic 划分为多个分区（Partition），Producer 生产的每条消息只会被发送到一个 Partition 中**！

每个 Partition 下可以配置 N 个 Replica，其中只能有 1 个  Leader Replica 和 N-1 个 Follower Replica。Producer 向 Partition 写入消息；

每条消息在 Partition 中的位置信息由唯一的 64 字节的位移（Offset）来表征。Consumer 可以置顶消费的位置信息，当消费者挂掉再恢复的时候，可以从消费位置继续消费。

## 副本机制（Replication）

Kafka 提供高可用的方式：

- 将不同的 Broker 分散运行在不同的机器上

> 多个 Broker 进程能够运行在同一台机器上，但更常见的做法是将不同的 Broker 分散运行在不同的机器上，这样如果集群中某一台机器宕机，即使在它上面运行的所有 Broker 进程都挂掉了，其他机器上的 Broker 也依然能够对外提供服务。

- **副本机制（Replication）**，可以保证数据的持久化或消息不丢失。

把相同的数据拷贝到多台机器上，而这些相同的数据拷贝在 Kafka 中被称为副本（Replica）。

Kafka 定义了两类 Replica：

1. Leader Replica，对外提供服务（与客户端程序进行交互）
2. Follower Replica，只是被动地追随领导者副本，**不能**与外界进行交互

Replica 的工作机制很简单，生产者总是向 Leader Replica 写消息；而消费者总是从 Leader Replica 读消息。至于 Follower Replica，它只提供数据冗余的功能，会向领导者副本发送请求，请求领导者把最新生产的消息发给它，这样它能保持与领导者的同步。

每个 Topic 可以配置 M 个 Partition，每个 Partition 可以配置 N 个 Replica；
而每个 Partition 层的 N 个 Replica 中只有一个 Leader Replica 和 N-1 个 Follower Replica，其中 Follower Replica 只是提供数据冗余的功能。

Client 只能和 Partition 的 Leader Replica 进行交互！

Kafka 消费者端实现高可用的手段是 Rebalance，当消费者离开消费组（比如重启、宕机等）时，它所消费的分区会分配给其他分区。

## Broker&Partition

> 从 Scale out 的角度，通过增加 Broker 可以带来更多的存储，建立更多的 Partition 可以把 IO 负载到更多的物理节点上，提高总的吞吐
> 从 Scale up 的角度，一个 Node（物理机）拥有越多的硬盘，也可以负载更多的 Partition，提高总的吞吐

如果节点数 == Partition 数，那么每个 Broker 存储该 Topic 的一个 Partition。

如果节点数 > Partition 数，如某 Topic 有 N 个 Partition，集群有 N+M 个 Broker，那么其中 N 个 Broker 存储该 Topic 的一个 Partition，剩下的 M 个 Broker 不会存储该 Topic 的 Partition 数据。

> 节点数 > Partition 数时，Partition 数量越大，吞吐率越高，且呈线性提升

如果节点数 < Partition 数，如某 Topic 有 N 个 Partition，集群中 Broker 数量小于 N 个，那么一个 Broker 存储该 Topic 的一个或多个 Partition。

> 节点数 < Partition 数，总吞吐量并未有所提升，甚至可能还会有所下降，原因可能是，不同 Broker 上的 Partition 数量不同，而消息被均匀分发到各个 Partition，造成各个 Broker 负载不同，无法最大化集群吞吐量

# 持久化

Kafka 是通过 Broker 来持久化数据的，依赖文件系统来存储和缓存消息。Kafka 的性能跟存储的数据量的大小无关， 所以将数据存储很长一段时间是没有问题的。

Kafka 使用消息日志（Log）来保存数据，一个日志就是磁盘上一个只能追加写（Append-only）消息的物理文件。

> 因为只能追加写入，故避免了缓慢的随机 I/O 操作，改为性能较好的顺序 I/O 写操作

由于日志会不断写入，所以需要定期删除消息来回收磁盘，Kafka 通过日志段机制（Log Segment）来定期删除。

在 Kafka 底层，一个日志又进一步细分成多个日志段，消息被追加写到当前最新的日志段中，当写满了一个日志段后，Kafka 会自动切分出一个新的日志段，并将老的日志段封存起来。

Kafka 在后台还有定时任务会定期地检查老的日志段是否能够被删除，从而实现回收磁盘空间的目的。

> Kafka 提供可配置的保留策略来删除旧数据（还有一种策略根据分区大小删除数据）。
> 如果将保留策略设置为两天，在 message 写入后两天内，它可用于消费，之后它将被丢弃以腾出空间。

## 稀疏索引

> 在 Kafka 的文件存储中，同一个 Topic 下有多个不同的 Partition，每个 Partition 都是一个目录。
> 而每个目录分为多个大小相等的 Segment File，Segment File 又由 index file 和 data file 组成，二者成对出现，后缀 ".index"、".log" 分别表示 Segment 索引文件和数据文件。
> 文件名为上一个 Segment 最后一条消息的 Offset。

Kafka 采用**稀疏索引**存储的方式，每隔一定子节的数据建立一条索引，并不是每一条消息都会保存索引。这样减少了索引文件大小，可以把索引映射到内存，降低了查询时的磁盘 IO 开销。

如果要查找一个指定 offset 的 message 时，通过 Segment 的文件名进行二分查找就能找到其归属的 Segment，再从 index 文件中找到对应到文件上的物理位置，就能拿出该 message。

> 在 Kafka 中定义了标准的数据存储结构，Partition 中的每条 Message 都包含了三个属性：
> - offset，表示 message 在当前 Partition 中的偏移量，唯一确定了 Partition 中的一条 message；
> - messageSize，表示 message 内容 data 的大小；
> - data，message 的具体内容。

假设要查找 offset=368772 的 message
1. 根据 offset 对文件列表进行二分查找，可以快速定位到具体文件
2. 假设定位到了 00000000368769.index、00000000368769.log，而在索引文件中有 `<3, 497>` 的一个元数据。其中 3 表示在数据文件中的第三个 message，对应全局 Partition 的第 368769+3=368772 个 message；497 表示该 message 的物理偏移地址为 497。
3. 再根据 messageSize 就可以从日志文件中顺序找到消息的具体位置。

Kafka 从 0.10.0.0 版本起，为分片日志文件增加了一个 .timeindex 的索引文件，可以根据时间戳定位消息。

# Questions

## 为什么 Kafka 的 Follower Replica 不对外提供读服务？

1. 使用场景，Kafka 的主要场景是在消息引擎，通常会涉及频繁的生产和消费过程，并非读多写少的场景，所以读写分离的方案并不适用；
2. Kafka 的副本机制是使用的异步消息拉取，会存在 follower 和 leader 的不一致，如果 follower 对外提供读服务，就需要引入 replica 的一致性问题；
3. 读写分离主要是为了减轻 leader 的压力，将读请求负载均衡到 follower，如果 Kafka 的分区相对均匀地分散在各个 broker 上，也可以达到负载均衡的效果，没必要使用读写分离增加实现复杂度。

> Replication 是在 Partition 上定义的，分区均匀分布的话，leader replica 也就均匀分布了，这样的话压力并不会集中在某个 broker 上，已经起到了负载均衡的效果。

https://www.zhihu.com/question/327925275/answer/705690755

