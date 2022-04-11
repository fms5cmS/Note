
Apache Kafka 是消息引擎系统，也是一个分布式流处理平台（Distributed Streaming Platform）！

Kafka 会将结构化的消息（Record）转换成**二进制的字节序列**来进行传输。

Topic：指发布订阅的对象，可以为每个业务、每个应用甚至是每类数据都创建专属的主题。

- Client 端
  - Producer：向 Topic 发布消息的客户端应用程序。Producer 通常会持续不断地向**一个或多个主题**发布消息
  - Consumer：订阅 Topic 消息的客户端应用程序。Consumer **可以同时订阅多个主题**的消息
- Server 端，由被称为 Broker 的服务进程构成
  - **一个 Kafka 集群由多个 Broker 组成，Broker 负责接收和处理客户端发送过来的请求，以及对消息进行持久化**

# Replica 和 Partition

## 副本机制（Replication）

Kafka 提供高可用的方式：

- 将不同的 Broker 分散运行在不同的机器上

多个 Broker 进程能够运行在同一台机器上，但更常见的做法是将不同的 Broker 分散运行在不同的机器上，这样如果集群中某一台机器宕机，即使在它上面运行的所有 Broker 进程都挂掉了，其他机器上的 Broker 也依然能够对外提供服务。

- **副本机制（Replication）**

把相同的数据拷贝到多台机器上，而这些相同的数据拷贝在 Kafka 中被称为副本（Replica）。

Kafka 定义了两类 Replica：

1. Leader Replica，对外提供服务（与客户端程序进行交互）
2. Follower Replica，只是被动地追随领导者副本，**不能**与外界进行交互

Replica 的工作机制很简单，生产者总是向领导者副本写消息；而消费者总是从领导者副本读消息。至于追随者副本，它只做一件事：向领导者副本发送请求，请求领导者把最新生产的消息发给它，这样它能保持与领导者的同步。

副本机制可以保证数据的持久化或消息不丢失。

Kafka 消费者端实现高可用的手段是 Rebalance（见消费模型部分）。

## 分区机制（Partitioning）

Kafka 的伸缩性（Scalability）问题是通过分区机制来解决的。

Kafka 的分区机制会**将每个 Topic 划分为多个分区（Partition），Producer 生产的每条消息只会被发送到一个 Partition 中**！

> 每个 Partition 是一组有序的消息日志。
> 
> 分区编号从 0 开始

## 说明

Replica 是在 Partition 这个层级定义的！

每个 Partition 下可以配置 N 个 Replica，其中只能有 1 个  Leader Replica 和 N-1 个 Follower Replica。Producer 向 Partition 写入消息；

每条消息在 Partition 中的位置信息由位移（Offset）来表征。分区位移总是从 0 开始。

# 三层消息架构

1. Topic 层，每个 Topic 可以配置 M 个 Partition，每个 Partition 可以配置 N 个 Replica；
2. Partition 层，每个 Partition 层的 N 个 Replica 中只有一个 Leader Replica 和 N-1 个 Follower Replica，其中 Follower Replica 只是提供数据冗余的功能；
3. 消息层，Partition 中包含了若干条消息，每条消息的 Offset 从 0 开始，依次递增。

Client 只能和 Partition 的 Leader Replica 进行交互！

# 持久化

Kafka 是通过 Broker 来持久化数据的。

Kafka 使用消息日志（Log）来保存数据，一个日志就是磁盘上一个只能追加写（Append-only）消息的物理文件。

> 因为只能追加写入，故避免了缓慢的随机 I/O 操作，改为性能较好的顺序 I/O 写操作

由于日志会不断写入，所以需要定期删除消息来回收磁盘，Kafka 通过日志段机制（Log Segment）来定期删除。

在 Kafka 底层，一个日志又进一步细分成多个日志段，消息被追加写到当前最新的日志段中，当写满了一个日志段后，Kafka 会自动切分出一个新的日志段，并将老的日志段封存起来。

Kafka 在后台还有定时任务会定期地检查老的日志段是否能够被删除，从而实现回收磁盘空间的目的。

# 消息模型

常见传输消息的两种模型是“点对点模型”和“发布/订阅模型”，Kafka 同时支持这两种模型。

- 点对点模型（Peer to Peer），同一条消息只能被下游的一个消费者消费，其他消费者则不能染指

Kafka 通过引入消费者组（Consumer Group）来实现 P2P 模型。

Consumer Group，指的是多个消费者实例共同组成一个组来消费一组主题。这组主题中的每个分区都只会被组内的一个消费者实例（Consumer Instance）消费，其他消费者实例不能消费它。

> 引入消费者组主要是为了提升消费者端的吞吐量。多个消费者实例同时消费，加速整个消费端的吞吐量（TPS）

Consumer Group 中的 Consumer Instance 之间可以彼此协作：假设组内某个实例挂掉了，Kafka 能够自动检测到，然后把这个 Failed 实例之前负责的分区转移给其他活着的消费者。这个过程就是 Kafka 中的“重平衡”（Rebalance）。

每个消费者在消费消息的过程中有个字段记录它当前消费到了分区的哪个位置上，这个字段就是消费者位移（Consumer Offset）。

# Questions

## 为什么 Kafka 的 Follower Replica 不对外提供读服务？

1. 使用场景，Kafka 的主要场景是在消息引擎，通常会涉及频繁的生产和消费过程，并非读多写少的场景，所以读写分离的方案并不适用；
2. Kafka 的副本机制是使用的异步消息拉取，会存在 follower 和 leader 的不一致，如果 follower 对外提供读服务，就需要引入 replica 的一致性问题；
3. 读写分离主要是为了减轻 leader 的压力，将读请求负载均衡到 follower，如果 Kafka 的分区相对均匀地分散在各个 broker 上，也可以达到负载均衡的效果，没必要使用读写分离增加实现复杂度。

> Replication 是在 Partition 上定义的，分区均匀分布的话，leader replica 也就均匀分布了，这样的话压力并不会集中在某个 broker 上，已经起到了负载均衡的效果。

https://www.zhihu.com/question/327925275/answer/705690755

