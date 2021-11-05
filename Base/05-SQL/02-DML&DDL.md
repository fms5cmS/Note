# DML

DML 数据操纵语言，针对**表的数据**进行增删改。

## 插入 insert

使用`values`来插入的方式支持插入多行，且支持子查询，而使用`set`插入的方式不支持 。

```sql
-- 表有几列就插入几个值
insert into table1 values(值1，值2，...)
-- 向特定列插入多条数据
insert into table1(column1,column2,...)
    values(值1，值2，...)(值1，值2，...)
-- 使用 set 插入数据
insert into table1 set column1=值1，...
```

插入检索到的信息 :

```sql
-- 将表2查到的列1、列2插入到表1中
insert into table1 select col1,col2 fromtable2
-- 将表2查到的列3、列4插入到表1的列1、列2中
insert into table1 (col1,col2...) select col3,col4 from table2
```

- 插入的值的类型要与列的类型一致或兼容
- 列数和值的个数必须一致
- 可以省略列名，默认所有列，而且列的顺序和表中列的顺序一致

## 修改 update

- 修改单表

```sql
update 表名 set 列名1 = xxx，列名2=xxx...[where语句]
```

示例：

```sql
-- 案例1：修改没有男朋友的女神的男朋友编号都为2号
UPDATE boys bo
RIGHT JOIN beauty b ON bo.id=b.boyfriend_id
SET b.boyfriend_id=2
WHERE bo.id IS NULL;
```

修改多表很少使用，故不列出。

## 删除 delete&truncate

- delete，多表的删除很少使用不再列出。单表的删除：

```sql
-- 删除表中筛选出来的内容
delete from 表名 where 筛选条件;
```

- truncate，删除表中全部数据。不可以加筛选条件

```sql
truncate table 表名;
```

**对比**：

1. `delete`可以加`where`条件，`truncate`不能加；
2. `truncate`删除，效率高一些；
3. 假如要删除的表中有自增长列，如果用`delete`删除后，再插入数据，自增长列的值从断点开始，而`truncate`删除后，再插入数据，自增长列的值从 1 开始；
4. `truncate`删除没有返回值，`delete`删除有返回值；
5. `truncate`删除不能回滚，`delete`删除可以回滚。

# DDL

DDL 数据定义语言，**针对表或库的结构进行增删改**

## 库的管理

- 创建：`create database 库名 if not exists gc`
- 修改：`rename/alter database`
  - 更改库的字符集：`alter database gc character set gbk`
- 删除：`drop database if exists gc`
- 查看：`show databases`

## 表的创建

```sql
CREATE TABLE  [IF NOT EXISTS] table_name(
  列名 类型（长度） [ 约束 ],
  列名 类型（长度） [ 约束 ],
  。。。
  列名 类型（长度） [ 约束 ] -- 这里不能有逗号
)ENGINE = InnoDB CHARACTER SET = utf8 COLLATE = utf8_general_ci ROW_FORMAT = Dynamic;
-- 表的约束，字符编码为 utf8，排序规则为 utf8_general_ci，代表对大小写不敏感，行格式为 Dynamic
```

建表时可以对表名和字段名加上反引号，避免与 MySQL 保留字段相同。

查看表的结构信息：`describe table_name` 或 `desc table_name`。

## 表的修改

这一节将所有的关键字大写，小写的为用户实际使用时要按需修改的。

改列名：`ALTER TABLE table1 CHANGE [COLUMN] oldName newName 类型`

```sql
ALTER TABLE book CHANGE COLUMN publishdate pubDate DATETIME;
```

改类型或约束：`ALTER TABLE table1 MODIFY [COLUMN] columnName 类型 [新约束]`

```sql
ALTER TABLE book MODIFY COLUMN pubdate TIMESTAMP;
```

增加列：`ALTER TABLE table ADD [COLUMN] columnName 类型 [约束]`

```sql
 ALTER TABLE author ADD COLUMN annual DOUBLE;
```

删除列：`ALTER TABLE table1 DROP [COLUMN] columnName`，不能用 `if exists`

```sql
ALTER TABLE author DROP COLUMN annual;
```

改表名：`ALTER TABLE oldTableName RENAME [TO] newTableName`

```sql
ALTER TABLE book_author RENAME TO author;
```

## 表的删除&复制

删除表：`drop table if exists tableName`

复制表结构：`CREATE TABLE copy LIKE author`

复制表结构+数据：`CREATE TABLE copy2 SELECT * FROM author`

复制部分字段：`CREATE TABLE copy4 SELECT id,au_name FROM author WHERE 0`

复制部分数据：`CREATE TABLE copy3 SELECT id,au_name FROM author WHERE nation='中国'`

注意：`WHERE` 语句中 0 代表 false，非 0 代表 true。

## 约束

约束：用于限制表中的数据，保证表中的数据的准确和可靠性。六大约束：

- 非空 `NOT NULL`：保证该字段的值不能为空。如：姓名
- 默认 `DEFAULT`：保证该字段有默认值。如：性别
- 主键 `PRIMARY KEY`：保证该字段的值具有唯一性，并且非空，一个表中最多有一个，虽然可以组合但不推荐。如：学号
- 唯一 `UNIQUE`：保证该字段的值具有唯一性，可以为空，一个表中可以有多个，虽然可以组合但不推荐。如：座位号
- 检查 `CHECK`：限制字段的条件。注意：mysql 不支持，但为了兼容，语法不报错。如：性别（只有男女）
- 外键 `FOREIGN KEY`：限制两表的关系，保证该字段的值必须来自于主表关联列的值；在从表添加外键，用于引用主表中某列的值。如：学生表的专业编号
  - 要求在从表设置外键关系
  - 从表的外键列的类型和主表的关联列的类型要求一致或兼容，名称无要求
  - 主表的关联列必须是一个 key（一般是主键或唯一）
  - 插入数据时，先插入主表，再插入从表；删除数据时，先删除从表，再删除主表

### 添加约束

|      | 位置     | 支持的约束类型       | 是否可以起约束名   |
| ---- | ------ | ------------- | ---------- |
| 列级约束 | 列的后面   | 语法都支持，但外键没有效果 | 不可以        |
| 表级约束 | 所有列的下面 | 默认和非空不支持，其他支持 | 可以（主键没有效果） |

```sql
CREATE TABLE 表名(
  字段名 字段类型 列级约束,
  字段名 字段类型,
  表级约束
)
```

- 建表时添加

```sql
CREATE TABLE major(         -- 该表用于外键的设置
  id INT PRIMARY KEY,
  majorName VARCHAR(20)
);

CREATE TABLE IF NOT EXISTS stuinfo(
  id INT PRIMARY KEY,      -- 主键
  stuname VARCHAR(20) NOT NULL UNIQUE, -- 非空、唯一
  sex CHAR(1),
  age INT DEFAULT 18, -- 默认（后面记得跟上默认值）
  seat INT UNIQUE, -- 唯一
  majorid INT,
  CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id)
  -- 外键起名：fk_当前表名_主表名
);

-- DESC无法查看外键
DESC stuinfo;

-- 查看stuinfo中的所有索引，包括主键、外键、唯一
SHOW INDEX FROM stuinfo;
```

上面的示例中除外键以外的约束都是使用的列级约束，也可使用表级约束：

```sql
DROP TABLE IF EXISTS stuinfo; #删除上面创建的表
CREATE TABLE stuinfo(
    id INT,
    stuname VARCHAR(20),
    gender CHAR(1),
    seat INT,
    age INT,
    majorid INT,

    CONSTRAINT pk PRIMARY KEY(id), -- 主键
    CONSTRAINT uq UNIQUE(seat), -- 唯一键
    CONSTRAINT ck CHECK(gender ='男' OR gender  = '女'), -- 检查，MySQL不支持
    CONSTRAINT fk_stuinfo_major FOREIGN KEY(majorid) REFERENCES major(id) -- 外键
);
SHOW INDEX FROM stuinfo;    -- 查看
```

---

- 修改表时添加约束

列级约束：`alter table 表名 modify column 字段名 字段类型 新约束`

表级约束：`alter table 表名 add [constraint约束名] 约束类型(字段名) [外键的引用]`

```sql
#1.先新建一个没有约束的表：
DROP TABLE IF EXISTS stuinfo;
CREATE TABLE stuinfo(
    id INT,
    stuname VARCHAR(20),
    gender CHAR(1),
    seat INT,
    age INT,
    majorid INT
)
DESC stuinfo;

#2.添加非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20)  NOT NULL;
#3.添加默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT DEFAULT 18;
#4.添加主键
-- ①列级约束
ALTER TABLE stuinfo MODIFY COLUMN id INT PRIMARY KEY;
-- ②表级约束
ALTER TABLE stuinfo ADD PRIMARY KEY(id);

#5.添加唯一
-- ①列级约束
ALTER TABLE stuinfo MODIFY COLUMN seat INT UNIQUE;
-- ②表级约束
ALTER TABLE stuinfo ADD UNIQUE(seat);

#6.添加外键
ALTER TABLE stuinfo ADD [CONSTRAINT fk_stuinfo_major] FOREIGN KEY(majorid) REFERENCES major(id);
```

### 删除约束

```sql
#1.删除非空约束
ALTER TABLE stuinfo MODIFY COLUMN stuname VARCHAR(20) NULL;  -- 也可以不写NULL，即为默认

#2.删除默认约束
ALTER TABLE stuinfo MODIFY COLUMN age INT ;

#3.删除主键
ALTER TABLE stuinfo DROP PRIMARY KEY;

#4.删除唯一
ALTER TABLE stuinfo DROP INDEX seat;

#5.删除外键
ALTER TABLE stuinfo DROP FOREIGN KEY fk_stuinfo_major;

SHOW INDEX FROM stuinfo; #查看
```

## 自增长列

又称标识列，可以不用手动的插入值，系统提供默认的序列值。

- 标识列必须和主键搭配吗？不一定，但要求是一个 key
- 一个表可以有几个标识列？至多一个！
- 标识列的类型只能是数值型
- 标识列可以通过`SET auto_increment_increment=3`;设置步长

MySQL 不支持设置自增长的起始值，但可以通过手动插入值，设置起始值。

- 创建表时设置标识列
  
  ```sql
  DROP TABLE IF EXISTS tab_identity;
  CREATE TABLE tab_identity(
      id INT  ,
      NAME FLOAT UNIQUE AUTO_INCREMENT,
      seat INT
  );
  
  SHOW VARIABLES LIKE '%auto_increment%'; -- 查看自增长变量
  ```

- 修改表时设置标识列
  
  ```sql
  ALTER TABLE tan_identity MODIFY COLUMN id INT PRIMARY KEY AUTO_INCREMENT;
  ```

- 修改表时删除标识列
  
  ```sql
  ALTER TABLE tan_identity MODIFY COLUMN id INT PRIMARY KEY ;
  ```
