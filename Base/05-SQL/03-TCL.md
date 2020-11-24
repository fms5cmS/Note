# 事务 transaction

事务（transaction）特点：原子性是基础，隔离性是手段，一致性是约束条件，持久性是目的！

- Atomicity（原子性）：一个事务内的所有操作是一个整体，要么全部成功，要么全失败；
- Consistency（一致性）：事务提交或回滚后，数据库的完整性约束不能被破坏；
- Isolation（隔离性）：一个事务在提交之前，对其他事务都是不可见的；
- Durability（持久性）：事务提交之后对数据的修改是持久性的

# 事务使用

事务的控制语句：

```sql
SET TRANSACTION -- 设置事务的隔离级别
START TRANSACTION 或 BEGIN  -- 显式开启事务
COMMIT -- 提交事务
SAVEPOINT -- 创建保存点
RELEASE SAVEPOINT -- 删除某个保存点
ROLLBACK 或者 ROLLBACK TO [SAVEPOINT] --回滚事务
```

连续 BEGIN，当开启了第一个事务，还未 COMMIT 时，会直接进行第二个事务的 BEGIN，这时数据库会隐式地 COMMIT 第一个事务，然后再进入到第二个事务。

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

- completion=0，默认情况
  - `COMMIT` 会提交事务，执行下一个事务时，还要使用 `START TRANSACTION` 或者 `BEGIN` 来开启。
- completion=1
  - `COMMIT` = COMMIT AND CHAIN，链式事务，即当提交当前事务后开启一个相同隔离级别的事务
- completion=2
  - `COMMIT` = COMMIT AND RELEASE，提交后，会自动与服务器断开连接

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

以上代码得到的结果为关羽、张飞。第二个 `BEGIN` 时隐式提交了第一个事务，并开启第二个事务，最后的 `COMMIT` 会强制提交，所以张飞的第一条成功执行插入语句被提交了。

# 隔离级别

多个事务同时执行时，由于隔离性的问题，可能出现脏读、不可重复读、幻读的问题。为了解决这些问题，就有了隔离级别，从低到高：（实现上，数据库中会创建一个视图，访问时以视图的逻辑结果为准。）

- 读未提交：允许事务读取未被其他事务提交的变更
  - 直接返回记录上的最新值，没有视图；
- 读已提交：只允许事务读取已被其他事务提交的变更
  - 该视图在每个 SQL 开始执行时创建
- 可重复读：一个事务执行过程中看到的数据总是与该事务启动时看到的数据一致
  - 该视图在事务启动时创建，整个事务存在期间都用这个视图
  - 在这个事务持续期间，禁止其他事务对这个字段进行更新
- 串行化：确保事务可以从一个表中读取相同的行。
  - 直接用加锁方式避免并行访问，性能很低

| 隔离级别（✓ 表示会发生该问题） | 脏读 | 不可重复读 | 幻读 |
| :----------------------------: | :--: | :--------: | :--: |
|  读未提交（Read Uncommitted)   |  ✓   |     ✓      |  ✓   |
|    读已提交(Read Committed)    |  ×   |     ✓      |  ✓   |
|   可重复读（Repeatable Read)   |  ×   |     ×      |  ✓   |
| 串行（序列）化（serializable)  |  ×   |     ×      |  ×   |

MySQL 支持 4 种事务隔离级别，默认的事务隔离级别是 `Repeatable Read`。 可以直接在 MySQL 中修改隔离级别，而不用在程序中修改。

Oracle 仅支持 2 种事务隔离级别：`Read committed`、`serializable`。默认的事务隔离级别是`Read committed`。

```sql
-- 查看当前连接的隔离级别
SELECT @@TX_ISOLATION;
-- 设置隔离级别
SET SESSION|GLOBAL TRANSACTION ISOLATION LEVEL 隔离级别;
```

