锁是计算机协调多个进程或线程并发访问某一资源的机制。数据库中，除了传统的计算机资源（CPU、RAM、I/O等）的争用外，数据也是一种供许多用户共享的资源，如何保证数据并发访问的一致性、有消息是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的重要因素。

数据库中锁的分类：

- 从对数据操作的类型分：读/写
  - 读锁（共享锁）：针对同一份数据，多个读操作可以同时进行而不会互相影响；
  - 写锁（排他锁）：当前写操作没有完成前，它会阻断其他写锁和读锁。
- 从对数据操作的粒度分：表锁、行锁

MySQL 中对锁的操作：

```sql
-- 1.查看表上加过的锁
show open tables;
-- 2.增加表锁
LOCK TABLE 表1 READ(WRITE)，表2 READ(WRITE)...;
-- 示例：给 mylock 表上读锁，book表上写锁
LOCK TABLE mylock READ,book WRITE;
-- 3.解锁
UNLOCK TABLES;
```

开销、加锁速度、死锁、粒度、并发性能只能就具体应用的特点来说哪种锁更合适。



# 表锁(偏读)

- 偏向 MyISAM 存储引擎，开销小，加锁快；
- 无死锁；
- 锁粒度大，发生锁冲突的概率最高，并发度最低。
- 读锁：

| 加读锁             | 被锁的表                                                   | 其他表         |
| ------------------ | ---------------------------------------------------------- | -------------- |
| 执行上锁操作的会话 | 只能查询，不能做其他操作                                   | 不能做任何操作 |
| 其他会话           | 只能查询，其他操作会发生**阻塞**，等待锁被释放才会继续执行 | 可执行任何操作 |

- 写锁：

| 加写锁             | 被锁的表                                   | 其它表         |
| ------------------ | ------------------------------------------ | -------------- |
| 执行上锁操作的会话 | 可执行任何操作                             | 不能做任何操作 |
| 其他会话           | 任何操作都被阻塞，等待锁被释放才会继续执行 | 可执行任何操作 |



## 示例

1. 建表

```sql
CREATE TABLE mylock(
	id INT NOT NULL PRIMARY KEY AUTO_INCREMENT,
	name VARCHAR(20)
)ENGINE=MyISAM;
```

2. 加读锁

```sql
LOCK TABLE mylock READ;
-- 如果要加写锁：
-- LOCK TABLE mylock WRITE;
```

3. 查看表的锁

```sql
SHOW OPEN TABLES;
-- 显示结果（这里仅列出了设置锁的表）：
Database	   Table		  In_use	Name_locked
myemployees	 mylock		     1		   0			-- 设置了读锁（共享锁）
myemployees	 locations  	 0		   0 			-- 没有设置锁
```



## 总结

MyISAM 执行查询语句前，自动给所有表加读锁；执行增删改操作前，自动给设计的表加写锁。

MySQL 的表级锁有两种模式：表共享读锁（Table Read Lock）；表独占写锁（Table Write Lock）。

| 锁类型 | 可否兼容 | 读锁 | 写锁 |
| ------ | -------- | ---- | ---- |
| 读锁   | 是       | 是   | 否   |
| 写锁   | 是       | 否   | 否   |

综合上表，所以对 MyISAM 表进行操作，会有以下情况：

1. 对 MyISAM 表的读操作（加读锁），不会阻塞其他进程对同一表的读操作，但会阻塞对同一表的写请求，只有当读锁释放后，才会执行其他进程的写操作；
2. 对 MyISAM 表的写操作（加写锁），会阻塞其他进程对同一表的读和写操作，只有当写锁释放后，才会执行其他进程的读写操作。

简而言之，**读锁会阻塞写，但不会阻塞读；写锁会把读和写都阻塞**。

MyISAM 的读写锁调度是写优先，故 MyISAM 不适合做写为主的引擎。因为写锁后，其他线程不能做任何操作，大量的更新会使查询很难得到锁，从而造成永远阻塞。

---

可以通过检查`table_locks_waited`和`table_locks_immediate`状态变量来分析系统上的表锁定。

```sql
SHOW STATUS LIKE 'table%';
```

`table_locks_waited`：出现表级锁定争用而发生等待的次数（不能立即获取锁的次数，每等待一次锁，值加一），此值高则说明存在较严重的表级锁竞争情况；

`table_locks_immediate`：产生表级锁定的次数，表示可以立即获取锁的查询次数，每立即获取锁，值加一。



# 行锁(偏写)

- 偏向InnoDB存储引擎，开销大，加锁慢；
- 会出现死锁；
- 锁定粒度最小，发生锁冲突的概率最低，并发度也最高。



## 示例

1. 建表

```sql
CREATE TABLE test_innodb_lock(
	a INT(11),
	b VARCHAR(16)
)ENGINE=INNODB;
```

2. 建索引

```sql
CREATE INDEX test_innodb_a_ind on test_innodb_lock(a);
CREATE INDEX test_innodb_lock_b_ind on test_innodb_lock(b);
```

3. 行锁操作（从左到右，从上往下）

| session1                                                     | session2                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `set autocommit=0`取消自动提交 ！                            | `set autocommit=0`取消自动提交！                             |
| `update test_innodb_lock set b='b3' where a=3;`更新数据（原来的数据b=a3）成功，但未提交 | 无操作                                                       |
| `select * from test_innodb_lock`查出的结果b=b3               | `select * from test_innodb_lock`查出的结果`b=a3`。           |
| `commit;`提交成功                                            | `select * from test_innodb_lock`查出的结果b=a3。如果session3没有取消自动提交的话，可以查出b=b3 |
| 无操作                                                       | `commit;`然后`select * from test_innodb_lock`查出的结果`b=b3`。 |
| `update test_innodb_lock set b='c3' where a=3;`修改成功但未提交 | `update test_innodb_lock set b='d3' where a=3;`阻塞          |
| `commit;`提交成功                                            | 上面的语句执行成功。`commit;`此时b=d3                        |
| `update test_innodb_lock set b='e3' where a=3;`修改成功但未提交 | `update test_innodb_lock set b='b5' where a=5;`修改成功但未提交 |
| `commit;`提交成功                                            | `commit;`提交成功                                            |

---

- 索引失效会导致行锁变表锁

| session1                                                     | session2                                                |
| ------------------------------------------------------------ | ------------------------------------------------------- |
| `set autocommit=0`取消自动提交 ！                            | `set autocommit=0`取消自动提交 ！                       |
| `update test_innodb_lock set a=41 where b=60;`这里b为**`varchar`类型，没有加引号导致索引失效，行锁变表锁**。更新成功未提交 | `update test_innodb_lock set b='b7' where a=7;`阻塞！！ |

- 如何锁定一行

```sql
-- 不需要取消自动提交
begin;
select * from test_innodb_lock where a=7 for update; -- 此时就会把a=7的记录行锁定
-- 此时，其他session再对这一行执行操作就会被阻塞
commit； -- 释放行锁
```



## 间隙锁

当用范围条件而不是相等条件检索数据，并请求共享或排他锁时，InnoDB 会给符合条件的已有数据记录的索引项加锁；对于键值在条件范围内但并不存在的记录，叫做”间隙（GAP）“。

InnoDB 也会对这个“间隙”加锁，这种锁机制就是所谓的间隙锁（Next-Key锁）。

间隙锁危害（先删除一条记录使得a列的数字不再连续，如删除a=2的记录）：

| session1                                                     | session2                                             |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| `set autocommit=0`取消自动提交 ！                            | `set autocommit=0`取消自动提交 ！                    |
| `update test_innodb_lock set b='sss' where a>1 and a<6;`更新成功，等待提交 | `insert into test_innodb_lock values(2,'aa');`阻塞！ |
| `commit;`                                                    | 上面的语句执行成功，`commit;`                        |

说明：

Query 执行过程中通过范围查找的话，会锁定整个范围内所有的索引键值，即使该键值并不存在。

间隙锁有一个比较致命的弱点，就是当锁定一个范围键值后，即使某些不存在的键值也会被无辜锁定，而造成在锁定时无法插入锁定键值范围内的任何数据。在某些场景下可能会对性能造成很大危害。



## 总结

InooDB 由于实现了行锁，虽然在锁定机制的实现方面所带来的性能损耗可能比表锁会更高一些，但在整体并发处理能力方面要远远优于 MyISAM 的表锁。当系统并发量较高时，InnoDB 的整体性能和 MyISAM相比会有较明显的优势。

但 InnoDB 的行锁同样有着脆弱的一面，当我们使用不当时，可能会让 InnoDB 的整体性能表现不仅无法高于 MyISAM，反而会更差。

---

通过检查`Innodb_row_lock`状态变量来分析系统上的行锁的争夺情况。

```sql
SHOW STATUS LIKE 'innodb_row_lock%';
```

| 状态变量                        | 含义                                 | 重要 |
| ------------------------------- | ------------------------------------ | ---- |
| `Innodb_row_lock_current_waits` | 当前正在等待锁定的数量               |      |
| `Innodb_row_lock_time`          | 从系统启动到现在锁定总时长           | ★★★  |
| `Innodb_row_lock_time_avg`      | 每次等待所花平均时间                 | ★★★  |
| `Innodb_row_lock_time_max`      | 从系统启动到现在等待最长的一次所耗时 |      |
| `Innodb_row_lock_waits`         | 系统启动后到现在总共等待的次数       | ★★★  |

当等待次数很高，且等待时间也不小时，就需要分析系统中为什么有这么多等待，然后根据分析结果指定优化计划。

---

- 优化建议

尽可能让所有数据检索都通过索引来完成，避免无索引导致行锁升级为表锁；

合理设计索引，尽量缩小锁的范围；

尽可能较少检索条件，避免间隙锁；

尽量控制事务大小，减少锁定资源量和时长；

尽可能低级别事务隔离。



# 页锁

- 开锁和加锁时间介于表锁和行锁之间；
- 会出现死锁；
- 锁定粒度介于表锁和行锁之间，并发度一般。

了解即可。



# for update

`for update`是一个悲观锁。使用时要注意：

1. 使用InnoDB引擎；
2. 使用时如果有明确主键，产生的是行锁；

```sql
-- 明确指定主键，且有结果集(即在表中存在要查询的数据)，行锁
select * from zzk_product where id='66' for update;
-- 明确指定主键，且无结果集，无lock
select * from zzk_product where id='-100' for update;
```

3. 如果没有明确的主键，则产生表锁。

```sql
-- 无主键，表锁
select * from zzk_product where name='iphone' for update;
-- 主键不明确，表锁
select * from zzk_product where id<>'66' for update;
select * from zzk_product where id like '66' for update;
```

