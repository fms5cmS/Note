Cache 的主要用途：高并发和高性能。使用缓存可能造成：

- 缓存与数据库双写不一致
- 缓存雪崩
- 缓存穿透
- 缓存并发竞争

Redis 高并发：主从架构，一主多从。单主用来写入数据，单机几万 QPS，多从用来查询数据，多个从实例可以提供每秒 10 万的 QPS。如果要容纳大量数据，可以使用 Redis 集群，而且用 redis 集群之后，可以提供可能每秒几十万的读写并发。

Redis 高可用：如果做主从架构部署，其实就是加上哨兵就可以实现了，任何一个实例宕机，自动会进行主备切换。

# NoSQL

当下的应用是 SQL + NoSQL 一起使用。

高并发的操作是不建议有关联查询的，会用冗余数据来避免关联查询。

NoSQL 数据库的四大分类：

- KV 键值： Memcache、Redis
- 文档型数据库(BSON 格式较多)：CouchDB、MongoDB (是基于分布式文件存储的数据库)
- 列存储数据库：Cassandra、HBase、分布式文件系统
- 图关系数据库：不是放图形的，放的是关系！如社交网络。Neo4J、InfoGrid

根据 CAP 原理，NoSQL 分为满足 CA、满足 CP、满足 AP 原则的三大类：

- CA：单点集群，满足一致性、可用性的系统，通常在可扩展性上不太强大
  - RDBMS(传统关系型数据库)
- CP：满足一致性、分区容错性的系统，通常性能不是很高
  - MongoDB、HBase、Redis
- AP：满足可用性、分区容错性的系统，通常可能对一致性要求低一些
  - CouchDB、Cassandra、DynamoDB

# 比较 Redis 和 Memcached

- Redis 支持服务器端的数据操作：Redis 相比 Memcached 来说，拥有更多的数据结构和并支持更丰富的数据操作，通常在 Memcached 里，你需要将数据拿到客户端来进行类似的修改再 set 回去。这大大增加了网络 IO 的次数和数据体积。在 Redis 中，这些复杂的操作通常和一般的 GET/SET 一样高效。
- 内存使用效率对比：使用简单的 key-value 存储的话，Memcached 的内存利用率更高，而如果 Redis 采用 hash 结构来做 key-value 存储，由于其组合式的压缩，其内存利用率会高于 Memcached。
- 性能对比：由于 Redis 只使用单核，而 Memcached 可以使用多核，所以平均每一个核上 Redis 在存储小数据时比 Memcached 性能更高。而在 100k 以上的数据中，Memcached 性能要高于 Redis，虽然 Redis 最近也在存储大数据的性能上进行优化，但是比起 Memcached，还是稍有逊色。
- 集群模式：memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是 redis 目前是原生支持 cluster 模式的，redis 官方就是支持 redis cluster 集群模式的，比 memcached 来说要更好
