# 事务 transaction

事务的四大特点（ACID）

- atomicity（原子性）表示一个事务内的所有操作是一个整体，要么全部成功，要么全失败；
- consistency（一致性）当事务提交或发生回滚后，数据库的完整性约束不能被破坏。
- isolation（隔离性）每个事务都是彼此独立的，不会受到其他事务的执行影响。也就是说一个事务在提交之前，对其他事务都是不可见的。
- durability（持久性）事务提交之后对数据的修改是持久性的
  - 持久性是通过事务日志来保证的。日志包括了回滚日志和重做日志。
  - 当通过事务对数据进行修改的时候，首先会将数据库的变化信息记录到重做日志中，然后再对数据库中对应的行进行修改。
  - 即使数据库系统崩溃，数据库重启后也能找到没有更新到数据库系统中的重做日志，重新执行，从而使事务具有持久性。

原子性是基础，隔离性是手段，一致性是约束条件，而持久性是我们的目的。

---

事务的控制语句：

```sql
SET TRANSACTION -- 设置事务的隔离级别
START TRANSACTION 或 BEGIN  -- 显式开启事务
COMMIT -- 提交事务
SAVEPOINT -- 创建保存点
RELEASE SAVEPOINT -- 删除某个保存点
ROLLBACK 或者 ROLLBACK TO [SAVEPOINT] --回滚事务
```

使用事务有两种方式：

- 隐式事务：事务没有明显的开启和结束的标记。如`insert`、`update`、`delete`；
- 显式事务：事务有明显的开启和结束的标记
  - 前提：**必须先设置自动提交功能为禁用，即 `set autocommit=0`，该设置仅针对当前事务有效**
  - 在 MySQL 中用 `show variables like 'autocommit'` 查看自动提交的值

```sql
SET autocommit=0;
START TRANSACTION;
DELETE FROM account WHERE id=25;
SAVEPOINT a; -- 设置保存点，只搭配回滚使用
DELETE FROM account WHERE id=28;
ROLLBACK TO a;  -- 回滚到保存点
-- COMMIT
```

MySQL 中有时会在事务开始之前设置 `SET @@completion_type = 1;`

completion_type 参数的作用：

- completion=0，这是默认情况。
  - 执行 `COMMI`T 时会提交事务，在执行下一个事务时，还需要使用 `START TRANSACTION` 或者 `BEGIN` 来开启。
- completion=1
  - `COMMIT` = COMMIT AND CHAIN，也就是开启一个链式事务，即当提交事务之后会开启一个相同隔离级别的事务
- completion=2
  - `COMMIT` = COMMIT AND RELEASE，也就是当提交后，会自动与服务器断开连接

```sql
CREATE TABLE test(name varchar(255), PRIMARY KEY (name)) ENGINE=InnoDB;
SET @@completion_type = 1;
BEGIN;
INSERT INTO test SELECT '关羽';
COMMIT;  -- 会开启一个链式事务，相当于在下一行写入了 START TRANSACTION 或 BEGIN
INSERT INTO test SELECT '张飞';
INSERT INTO test SELECT '张飞';
ROLLBACK;
SELECT * FROM test; -- 结果只有 关羽
```

在 MySQL 中，如果是连续 `BEGIN`，开启了第一个事务，还没有进行 `COMMIT` 提交，而直接进行第二个事务的 `BEGIN`，数据库会隐式的帮助 `COMMIT` 第一个事务，然后进入到第二个事务。

```sql

DROP TABLE IF EXISTS test;
CREATE TABLE test(name varchar(255), PRIMARY KEY (name)) ENGINE=InnoDB;
BEGIN;
INSERT INTO test SELECT '关羽';
BEGIN;
INSERT INTO test SELECT '张飞';
INSERT INTO test SELECT '张飞';
COMMIT; -- 强制提交，会把这个事务中成功执行的部分进行提交。
SELECT * FROM test;
```

以上代码得到的结果为关羽、张飞。第二个 `BEGIN` 时隐式提交了第一个事务，并开启第二个事务，最后的 `COMMIT` 会强制提交，所以张飞的第一条成功执行插入语句被提交了，

# 隔离级别

对于同时运行的多个事务，当这些事务访问数据库中相同的数据时，如果没有采取必要的隔离机制，会导致并发问题：

- 脏读：读到了其他事务还没有提交的数据
  - 有两个事务 T1、T2，T1 读取了已经被 T2 更新但还没有被提交的字段，之后，若 T2 回滚，T1 读取的内容就是临时且无效的。即读到了其他事物修改了的数据；
- 不可重复读：对某数据进行读取，发现**两次读取的结果不同**
  - 有两个事务 T1、T2，T1 读取了一个字段，然后 T2 修改/删除了该字段，之后，T1 再次读取同一个字段，值就不同了
  - 重点在 UPDATE 和 DELETE
- 幻读：读到了其他事务**新增**的数据
  - 有两个事务 T1、T2，T1 从一个表中读取了一个字段，然后 T2 在该表中插入一些新的行，之后，如果 T1 再次读取同一个表就会多出几行
  - 重点在 INSERT
- 更新丢失：
  - 当两个事务选择同一行，然后基于最初选定值更新该行时，由于每个事务都不知道其他事务的存在，就会发生丢失更新问题——最后的更新覆盖了其他事务所做的更新。
  - 如果在一个程序员完成并提交事务之前，另一个程序员不能访问同意文件，即可避免此问题；

脏读要一定避免！

事务隔离级别从低到高：

- 读取未提交（Read Uncommitted)：允许事务读取未被其他事务提交的变更
  - 这种情况下查询是不会使用锁的，脏读、不可重复读和幻读都可能会出现
- 读取已提交(Read Committed)：只允许事务读取已被其他事务提交的变更
  - 可以避免脏读，但不可重复读和幻读仍会出现
  - 要避免不可重复读或者幻读，就需要我们在 SQL 查询的时候编写带加锁的 SQL 语句
- 可重复读（Repeatable Read)：确保事务可以多次从一个字段中读取相同的值，在这个事务持续期间，禁止其他事务对这个字段进行更新
  - 可以避免脏读和不可重复读，但幻读仍存在。
- 序列化（serializable)：确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作，多数并发问题都可避免，但性能十分低下。

| 隔离级别（✓ 表示会发生该问题） | 脏读 | 不可重复读 | 幻读 |
| :----------------------------: | :--: | :--------: | :--: |
| 读取未提交（Read Uncommitted)  |  ✓   |     ✓      |  ✓   |
|   读取已提交(Read Committed)   |  ×   |     ✓      |  ✓   |
|   可重复读（Repeatable Read)   |  ×   |     ×      |  ✓   |
| 串行（序列）化（serializable)  |  ×   |     ×      |  ×   |

Oracle 支持 2 种事务隔离级别：`Read committed`、`serializable`。默认的事务隔离级别是`Read committed`；

MySQL 支持 4 种事务隔离级别，默认的事务隔离级别是 `Repeatable Read`。 可以直接在 MySQL 中修改隔离级别，而不用在程序中修改。

每启动一个 MySQL 程序，就会获得一个单独的数据库连接，每个数据库连接都有一个全局变量`@@tx——isolation`，表示当前的事务隔离级别，MySQL 默认的隔离级别是`Repeatable Read`。

```sql
-- 查看隔离级别
SELECT @@TX_ISOLATION;
-- 设置隔离级别
SET SESSION|GLOBAL TRANSACTION ISOLATION LEVEL 隔离级别;
```
