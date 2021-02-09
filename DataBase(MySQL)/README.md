极客时间[《MySQL实战45讲》](https://time.geekbang.org/column/intro/139)笔记！

在 MySQL 数据表设计中，一般有这么一个原则：如果存在大字段数据，而且它一般不变，应该将不变的字段单独放入另外一个表中。

MySQL 的配置文件：Windows 中是 my.ini；Linux 中是`/etc/my.cnf`，主要的配置：

- 二进制日志 log-bin：主要用于主从复制。
- 错误日志 log-error：默认关闭，记录严重的警告和错误信息，每次启动和关闭的详细信息等。
- 查询日志 log：默认关闭，记录查询的 sql 语句，如果开启会降低 mysql 的整体性能，因为记录日志也需要消耗系统资源。
- 数据文件：
    - .frm 文件：存放表结构
    - .myd 文件：存放表数据
    - .myi 文件：存放表索引

注意：mysqld 是服务端程序；mysql 是命令行客户端程序。

CentOS 7 下 MySQL 的 YUM 安装方式：可以查看：[官方文档](https://dev.mysql.com/doc/mysql-yum-repo-quick-guide/en/)

通过 `select version();` 查询 MySQL 的版本；`show variables like 'character%'`查看字符集

MySQL 是典型的 C/S（Client/Server）架构，服务端程序使用的 mysqld。基本架构示意图：

![逻辑架构](../images/MySQL-logic.png)

- Server 层：涵盖 MySQL 大多数核心服务功能及所有内置函数，跨存储引擎的功能都在这一层实现，如存储过程、触发器、视图等；
    - 不同的存储引擎共用一个 Server 层
- 存储引擎层：负责数据存储和提取。MySQL 的存储引擎采用插件形式！
  - InnoDB：MySQL 5.5 后默认的存储引擎，最大的特点是支持事务、行级锁定、外键约束等；
  - MyISAM：MySQL 5.5 之前的默认存储引擎，不支持事务，也不支持外键，最大的特点是速度快，占用资源少；
  - Memory：使用系统内存作为存储介质，以便得到更快的响应速度。
    - 如果 mysqld 进程崩溃，则会导致所有的数据丢失，因此只有当数据是临时的情况下才使用 Memory 存储引擎
  - NDB：也叫 NDB Cluster，主要用于 MySQL Cluster 分布式集群环境，类似于 Oracle 的 RAC 集群
  - Archive：有很好的压缩机制，用于文件归档，在请求写入时会进行压缩，所以经常用来做仓库

注意：数据库的设计在于表的设计，而 MySQL 中每个表的设计都可以采用不同的存储引擎！

# 存储引擎

MySQL 中存储引擎的主要作用是负责数据的存储和提取。用的最多的存储引擎：`innodb`、`myisam`、`memory` 等。其中`innodb`支持事务，而`myisam`、`memory`等不支持事务。

```sql
-- 查看MySQL所支持的的存储引擎
show engines;
-- 查看MySQL当前默认的存储引擎
show variables like '%storage_engine%';
```

![命令查看存储引擎](../images/MySQL-engines.png)

注：图中的 TokuDB 存储引擎需要单独安装。



## InnoDB 和 MyISAM

MySQL 的默认存储引擎 InnoDB 使用的是 B+ 树来存储数据。

主要的两种存储引擎的区别：

|        | MyISAM                                                       | InnoDB                                                       |
| ------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 主外键 | 不支持                                                       | 支持                                                         |
| 事务   | 不支持                                                       | 支持                                                         |
| 行表锁 | 表锁<br>即使操作一条记录也会锁住整个表。<br>不适合高并发操作。 | 行锁。<br>操作时只锁住某一行，不对其它行有影响。<br/>**适合高并发操作**。 |
| 缓存   | 只缓存索引，不缓存真实数据                                   | 同时缓存索引、真实数据，对内存要求较高，<br>且内存大小对性能有决定性影响 |
| 表空间 | 小                                                           | 大                                                           |
| 关注点 | 性能                                                         | 事务                                                         |
| 日志   | redo log 和 binlog                                           | binlog                                                       |

InnoDB 存储引擎还有个“自适应 Hash 索引”的功能，就是当某个索引值使用非常频繁的时候，它会在 B+ 树索引的基础上再创建一个 Hash 索引，这样让 B+ 树也具备了 Hash 索引的优点。

InnoDB 三大关键特性：插入缓冲（Insert Buffer）、二次写(Double Write)、自适应 Hash。

## TokuDB

[TokuDB](https://www.percona.com/doc/percona-tokudb/index.html) 也支持事务，有着出色的**数据压缩功能**，完全兼容 ACID 和 MVCC。如果数据写多读少，且数据量较大，使用 TokuDB 可以节省空间成本，并大幅度降低存储使用量和IOPS开销，不过相应的会增加 CPU 的压力。

压缩：TokuDB 会压缩磁盘上的所有数据（包括索引），压缩后通过降低存储需求来减少成本，可以为额外的索引释放磁盘空间，从而提升查询性能，同时，从磁盘中读取和写入磁盘的数据减少，也会提高性能。

快速插入和删除（**Fast Insertions and Deletions**）：TokuDB 采用了叫做分形树（Fractal Tree）的索引结构。

消除从库延迟（**Eliminates Slave Lag**）：得益于 Fractal Tree 索引的特性，TokuDB 的 slave 端能在几乎没有读 IO 的情况下来复制 master 端变更；唯一性检查也可以仅在 master 端进行，而在 slave 端跳过。基于行（row）的复制可以确保在 binlog 中捕获到 row 变化前后的信息（原文为 images），因此 TokuDB slave 端可以利用 Fractal Tree 索引的功能，并绕过传统的 读取-修改-写入 行为。这种 “Read Free Replication” 确保了 slave 端复制不会滞后于 master 端，可用于 read scaling、备份、灾难恢复，且无需分片、昂贵的硬件或对可复制内容进行限制。

在线索引创建（**Hot Index Creation**）：TokuDB 允许在对表进行插入、查询操作的同时向该表添加索引。这也就意味着当添加索引时 MySQL 可以无阻塞的执行查询、插入操作，消除索引修改原本所需要的停机时间。

在线修改列（**Hot Column Addition, Deletion, Expansion and Rename**）：TokuDB 允许在对表进行插入、查询的同时，向表中增加新的列，删除已有的列，对 char、varchar、varbinary、int 类型的列扩充，对已有的列名修改。

在线备份（**Online (Hot) Backup**）：TokuDB 可以在不停机的情况下创建在线数据库的备份。

聚集索引及其他索引提升（**Clustering Keys and Other Indexing Improvements**）：TokuDB 的表是聚集在主键上的，TokuDB 也支持二级聚集索引（clustering secondary keys），二级聚集索引在大范围查询时有更好的性能。TokuDB 的索引最多支持 32 列，且自增列可以在任意的索引及索引内的任意位置。

更少的碎片（**Less Aging/Fragmentation**）：TokuDB 可以运行更长的时间，而不需要进行通常的 dump/reload 或 `OPTIMIZE TABLE` 操作来恢复数据库性能。其关键在于，在磁盘上以 Fractal Tree 存储数据。因为默认情况下，相比于 InnoDB 16KB 的大小，Fractal Tree 会以 4MB 的数据块（预压缩）来存储数据，TokuDB 避免数据库混乱（database disorder）的能力比 InnoDB 强 250 倍。可以参考：[Avoiding Fragmentation with Fractal Trees](https://www.percona.com/blog/2010/11/17/avoiding-fragmentation-with-fractal-trees/)

批量加载器（**Bulk Loader**）：TokuDB 使用并行加载器来创建表和离线索引，该并行加载器会使用多核来快速创建离线表和索引。

功能齐全（**Full-Featured Database**）：TokuDB 支持完全符合 ACID 的事务、MVCC、序列化事务级别、行锁以及 XA。

锁诊断（**Lock Diagnostics**）：TokuDB 提供了诊断锁和死锁的工具。

进程追踪（**Progress Tracking**）：添加索引时运行 `show processlist` 会提供已处理行数的状态，查询、插入、删除、更新操作时也可以显示。这个信息在判断操作执行耗时时是很有帮助的。

快速恢复（**Fast Recovery**）：TokuDB 支持快速恢复，通常在 1min 内。

# 数据导入/出

## 导出

- into outfile 导出：

在 my.ini 文件中设置`secure-file-priv=""`，就可以将导出文件放置在任意位置。

```sql
select * from user
	into outfile '/tmp/user.csv' -- 指定导出目录和文件名(该文件必须是不存在的)
	fields terminated by ','  -- 定义字段间的分隔符
	optionally enclosed by '"'  -- 定义包围字段的字符（数值型字段无效）
	lines terminated by '\r\n'; -- 定义每行的分隔符
```

注意：输出不能是一个已存在的文件，防止文件数据被篡改。

---

- 导出表作为原始数据：

`mysqldump`是 MySQL 用于转存储数据库的实用程序。主要产生一个 SQL 脚本，其中包含从头重新创建数据库所必需的命令 `CREATE TABLE` `INSERT`等。

使用`mysqldump`导出数据需要使用`--tab`选项来指定导出文件指定的目录，该目标必须是可写的。

在 MySQL 的安装目录下的 bin 目录进入命令行（不要进入数据库终端），执行：

```sql
mysqldump -u用户名 -p 数据库 要导出的表 > 文件目录和文件名
-- 如：mysqldump -uroot -p ssm_crud tbl_emp > d:\s.sql
```

如果要导出整个数据库的数据：`mysqldump -u用户名 -p 数据库名 > data.txt`

如果要导出所有数据库：`mysqldump -u用户名 -p --all-databases > data.txt`

## 导入

- mysql 命令导入

```sql
mysql -u用户名 -p < 要导入的数据库数据
-- 如：mysql -uroot -p < s.sql
```

---

- source 命令导入

登录到数据库终端——>创建数据库——>使用该数据库——>输入：`source 文件`。

如：`source d:\s.sql`

---

- load data 命令导入

```sql
load data [local] infile '文件' into table 表名;
-- 使用了 local 关键字，则从客户主机上按路径读取文件
-- 不使用，则文件在服务器上按路径读取文件
```

也可以像`into outfile`命令那样指定分隔符，使用相同：

- `fields terminated by ‘字段间分隔符’`：定义字段间的分隔符
- `lines terminated by ‘行间分隔符’`：定义每行的分隔符

`LOAD DATA`默认情况下是按照数据文件中列的顺序插入数据的，如果数据文件中的列与插入表中的列不一致，则需要指定列的顺序。如，在数据文件中的列顺序是 a,b,c，但在插入表的列顺序为 b,c,a，则数据导入语法如下：

```sql
LOAD DATA LOCAL INFILE 'dump.txt' INTO TABLE mytbl (b, c, a);
```

---

- mysqlimport 命令导入

在 MySQL 的安装目录下的 bin 目录下进入命令行(不用进数据库终端)，执行：

```sql
mysqlimport -u用户名 -p --local 数据库名 文件
```

可使用`--column`选项设置列的顺序，如：

```shell
mysqlimport -u root -p --local --columns=b,c,a database_name dump.txt
```
