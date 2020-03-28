# JDBC

说明：仅仅是入门笔记，视频：尚硅谷视频

# 1.认识

JDBC：Java DataBase Connectivity 

需要引入的jar包：

```xml
<!-- 依赖的mysql的jar包 -->
<dependency>
  <groupId>mysql</groupId>
  <artifactId>mysql-connector-java</artifactId>
  <version>6.0.6</version>
</dependency>
```

- 访问数据库的流程：
  - 1.加载一个Driver驱动    
  - 2.创建数据库连接（Connection）
  - 3.创建SQL命令发送器Statement  ，通过Statement 发送SQL命令并得到结果  
  - 4.处理结果（select语句）  
  - 5.关闭数据库资源 

# 2.开始

## 加载驱动

加载 JDBC 驱动是通过反射来完成的：

```java
//不同的数据库管理系统（DBMS）使用的驱动程序不同
Class.forName("com.mysql.jdbc.Driver"); //这里使用MySql的JDBC驱动程序。
```

## 创建数据库连接

连接对象内部其实包含了Socket对象，是一个远程连接。比较耗时 。开发中，为了提高效率，会使用连接池来管理对象。

- 与数据库建立连接是通过调用方法`DriverManager`类的`getConnection()`方法来完成的。

  ```java
  String url = "jdbc:mysql://localhost:3306/testjdbc";
  String user = "root"  //数据库的用户名
  String password = "123456"   //数据库的密码
  Connection conn = DriverManager.getConnection (url, user, password )
  conn.setAutoCommit(false); //JDBC中默认为true会自动提交，修改为false后需要手动提交
  con.commit()//手动提交
  ```

- 上面的代码耦合度较高，可以将数据库连接所需的信息抽取为配置文件 db.properties

  ```properties
  driver=com.mysql.cj.jdbc.Driver          这里将驱动也抽取出来了
  url=jdbc:mysql://localhost:3306/testjdbc
  username=root
  password=132128
  ```

  ```java
  //1.读取jdbc.properties文件
  Properties pro = new Properties();
  pro.load(getClass().getClassLoader().getResourceAsStream("db.properties"));
  //2.准备获取连接的字符串
  String driverClass = pro.getProperty("driver");
  String jdbcUrl = pro.getProperty("url");
  String user = pro.getProperty("username");
  String password = pro.getProperty("password");
  //3.加载驱动
  Class.forName(driverClass);
  //4.连接
  Connection connection = DriverManager.getConnection(jdbcUrl, user, password);
  ```

- ==注意：==

  - 1.在MySQL 6.0（指的是导入的 jar 包）以后，使用了新的驱动`com.mysql.cj.jdbc.Driver， `这个驱动会通过SPI（Service Provider Interface）自动加载，通常不需要人工手动加载。 所以==可以不再显式地加载驱动。==

  - 2.建立连接时的 url 中需要==增加时区信息==：增加 serverTimezone 属性并设置时区值，测试UTC（世界标准时间）和GMT（格林威治时间）都可以。 

  - 3.不推荐不使用服务器身份验证来建立SSL连接（Secure Sockets Layer 安全套接层 )。 

    - 如果未明确设置，MySQL 5.5.45+, 5.6.26+ and 5.7.6+版本默认要求建立SSL连接。为了符合当前不使用SSL连接的应用程序，`verifyServerCertificate`属性设置为’false’。 
    - 如果你不需要使用SSL连接，你需要通过设置`useSSL=false`来显式禁用SSL连接。如果你需要用SSL连接，就要为服务器证书验证提供信任库，并设置`useSSL=true`。

  - url 的参数之间使用 & 分开，所以 url 为：

    ```properties
    url=jdbc:mysql://localhost:3306/mybatis?serverTimezone=UTC&useSSL=false
    ```

  - 可以参照：<https://blog.csdn.net/booloot/article/details/76223004> 

## 创建 Statement

Statement 对象用于将 SQL 语句发送到数据库中，或者理解为执行sql语句有三种 Statement对象： 

1. `Statement`：用于执行**不带参数**的简单SQL语句（处理参数很麻烦，会发生SQL注入漏洞）； 
2. `PreparedStatement`（从 `Statement `继承。更常用）：用于执行带或不带参数的预编译SQL语句；
3. `CallableStatement`（从`PreparedStatement `继承）：用于执行数据库存储过程的调用。 


这里仅介绍了常用的`PreparedStatement`对象的使用：以下所有的 ps 指的就是该对象的实例

优点：会对SQL语句进行预编译，以减轻数据库服务器的压力 、代码可读性强、安全性高，可以防止SQL注入 

```java
// ?是占位符，需要的时候传入参数，只在PreparedStatement中使用，可以防止SQL注入,必须和setXxx()一起使用
String sql = "insert into  student(uname,age) values(?,?);";
PreparedStatement ps = con.prepareStatement(sql);//获取Prepared Statement对象
 //利用setXxx()传入参数，注意：参数索引从1开始计算。
ps.setString(1, "Nanjo");  //将第一个参数设置为String类型，值为“Nanjo”
ps.setInt(2, 17);

//也可以不管形式直接用setObject()传入参数
//ps.setObject(1, "fripSide");
//ps.setObject(2, "17");
ps.execute();    //执行操作
```

`PreparedStatement`对象的执行方法：

```java
ResultSet executeQuery ()      //返回单结果集，通常用于SELECT语句
boolean execute ()             //返回布尔值，通常用于insert，update，delete语句
int executeUpdate ()     //返回操作影响的行数，通常用于insert，update，delete语句
```

- 注意:如果要传入的 SQL 语句中有时间，传入的是`java.sql.Date()`方法：都是无参的方法。 

### SQL注入

正确的SQL语句必须要有正确的用户名和密码，如： 

```sql
SELECT * FROM users WHERE username = 'Tom' AND password = '123456'
```

而在使用`Statement`拼写SQL语句时，如果故意在输入用户信息时写成：

用户：`m' OR password = `     ； 密码：`OR '1'='1`        此时，使用`Statement`拼成的 SQL语句为：

```sql
SELECT * FROM users WHERE username = 'm' OR password = 'AND password =' OR '1'='1'
```

由于最后的 `'1'='1'`  一定为true，所以此时虽然没有输入正确的用户和密码，但还是查询成功了。

## 处理数据

`ResultSet`对象是`executeQuery()`方法的返回值，它被称为结果集，它代表符合SQL语句条件的所有行，并且它通过一套`getXXX(index)`或`getXXX(columnName)`方法（这些get方法可以访问当前行中的不同列）提供了对这些行中数据的访问。 

以下所有的 rs 都是 该对象的实例。

`ResultSet`里的数据一行一行排列，每行有多个字段，且有一个记录指针，指针所指的数据行叫做当前数据行，我们只能来操作当前的数据行。我们如果想要取得某一条记录，就要使用`ResultSe`t的`next()`方法 ,如果我们想要得到`ResultSet`里的所有记录，就应该使用while循环。 

`ResultSet`对象自动维护指向当前数据行的游标。每调用一次`next()`方法，游标向下移动一行。  

初始状态下记录指针指向第一条记录的前面，通过`next()`方法指向第一条记录。循环完毕后指向最后一条记录的后面 

```java
ResultSet rs = ps.executeQuery();      //接收结果集
//遍历输出结果集             
while(rs.next()) {
    //将结果集中第一列、第二列、第三列的数据输出
    //这里也可以用列名来取数据
    System.out.println(rs.getInt(1)+"---"+rs.getString(2)+"---"+rs.getInt(3));
}
/**
* ResultSet提供的检索不同类型字段的方法：
* getString()：获得数据库中varchar、char等类型的对象
* getFloat()：获得数据库中float类型的对象
* getDate()：获得数据库中Date类型的对象
* getBoolean()：获得数据库中Boolean类型的对象
*/
```

### 处理元数据

Java 通过 JDBC 获得连接后得到一个`Connection`对象，可以从这个对象获取有关数据库管理系统的各种信息，包括数据库中的各个表、表中的各个列、数据类型、触发器、存储过程等各方面的信息。根据这些信息，JDBC可以访问一个实现并不了解的数据库； 

- `DatabaseMetaData`类，描述数据库的元数据对象，而`DatabaseMetaData`对象是在`Connection`对象上获得的。 了解即可。

  - 方法见下:

    ```java
    getURL()  //返回一个String对象，代表数据库的URL
    getUserName（） //返回连接当前数据库管理系统的用户名
    isReadOnly（） //返回一个boolean值，只是数据库是否只允许读操作
    getDatabaseProductName（）  //数据库的产品名称
    getDatabaseProductVersion（） //数据库的版本号
    getDriverName（）  //返回驱动程序的名称
    getCatalogs() //得到MySQL中有哪些数据库，返回的是ResultSet，可能有多行，但都只有一列
    getDriverVersion（） //返回驱动程序的版本号
    ```

- ==ResultSetMetaData类==：描述结果集的元数据可用于获取关于`ResultSet`对象中列的类型和属性信息的对象。！！！！！ 

  - 获取该对象：调用 `ResultSet` 的 `getMetaData()` 方法  

  - 方法：

    ```java
    getColumnName(int column)：获取指定列的名称
    getColumnCount()：返回当前ResultSet对象中的列数（即SQL语句中包含的列数）
    getColumnLabel(int column)：获取指定列的别名，索引从 1  开始
    getColumnTypeName(int column)：检索指定列的数据库特定的类型名称
    getColumnDisplaySize(int column)：只是指定列的最大标准宽度，以字符为单位
    isNullable(int column)：只是指定列中的值是否可以为null
    isAutoIncrement(int column)：只是是否自动为指定列进行编号，这样这些列仍然是只读的
    ```

  举例：

  ```java
  //1.得到ResultSetMetaData对象
  ResultSetMetaData rsmd = resultSet.getMetaData();
  //2.打印每一列的列名
  for(int i = 0;i < rsmd.getColumnCount();i++) {
      System.out.println(rsmd.getColumnLabel(i+1));
  }
  ```

### 获取主键

```java
//使用重载的 prepareStatement(String sql, int columnIndexes[]) 方法生成PrepareStatement对象
ps = conn.prepareStatement(sql, Statement.RETURN_GENERATED_KEYS);
//执行sql语句并获取结果集
//通过 getGeneratedKeys() 方法获取新生成的包含主键的ResultSet对象
//在ResultSet中只有一列GENERATED_KEY，用于存放新生成的主键值
rs = ps.getGeneratedKeys();
if(rs.next()){
    System.out.println(rs.getObject(1));
}
```

## 关闭资源

遵循：`ResultSet`-->`Statement`-->`connection`这样的关闭顺序，一定要三个分开写，即写三个try-catch块 。

因为`Statement`和`ResultSet`是需要连接才可以使用的，所以在使用结束之后有可能其他的`Statement`还需要连接，所以不能先关闭`Connection`。 

## 特殊数据插入&获取

### 时间类型

- `java.util.Date`

  - 子类：`java.sql.Date` 表示年月日 （常用）
  - 子类：`java.sql.Time` 表示时分秒
  - 子类：`java.sql.Timestamp `表示年月日时分秒（常用） 

- 时间类型的插入：

  ```java
  ps1 = con.prepareStatement("insert into student(id,uname,regtime,lastlogintime) values(?,?,?,?)");
  ps1.setObject(1, 1);
  ps1.setObject(2, "Yoshino");
               
  java.sql.Date date = new java.sql.Date(132128);
  Timestamp t = new Timestamp(System.currentTimeMillis());
  //如果需要指定日期，可以使用Calendar、DateFormate
               
   ps1.setDate(3, date);
   ps1.setTimestamp(4, t);
  ```

- 日期比较处理

  - 插入随机日期：可以借助 `Random`类产生随机数
  - 取出指定日期范围的记录

  ```java
  //去除指定日期范围内的记录，自定义函数str2long将指定格式的日期字符串转成long类型的数字
  //如果要取类型为timeStamp的数据，只要将Date类型更换就好，因为timeStamp创建对象也是用long型参数的
  ps = con.prepareStatement("select * from student where regtime>? and regtime<?;");
               
  java.sql.Date start = new java.sql.Date(str2Long("1969-2-2 10:35:20")); //自定义函数
  java.sql.Date end = new java.sql.Date(str2Long("2018-5-2 10:35:20"));
               
  ps.setObject(1, start);
  ps.setObject(2, end);
               
  rs = ps.executeQuery();
  while(rs.next()) {
     System.out.println(rs.getInt("id")+"--"+rs.getString("uname")+"--"+rs.getDate("regtime"));
  }
  ```

  ```java
  //str2long将指定格式的日期字符串转成long类型的数字
  public static long str2Long(String dateStr) {
           DateFormat format = new SimpleDateFormat("yyyy-MM-dd hh:mm:ss");
           try {
               return format.parse(dateStr).getTime();
           } catch (ParseException e) {
               e.printStackTrace();
               return 0;
           }
      }
  ```

### CLOB

- CLOB（Character Large Object） 
  - 用于存储大量的文本数据 
  - 大字段有些特殊，不同数据库处理的方式不一样，大字段的操作常常是以流的方式来处理的。而非一般的字段，一次即可读出数据。 
- Mysql中相关类型：
  - TINYTEXT最大长度为$255(2^8–1)$字符的TEXT列。
  - TEXT[(M)]最大长度为$65,535(2^{16}–1)$字符的TEXT列。
  - MEDIUMTEXT最大长度为$16,777,215(2^{24}–1)$字符的TEXT列。
  - LONGTEXT最大长度为$4,294,967,295$或$4GB$$(2^{32}–1)$字符的TEXT列。 

```java
//插入
ps = con.prepareStatement("insert into student(uname,myinfo) values(?,?)");
ps.setString(1, "Nanjo");
//用流的方式处理
//1.将文本文件的内容放入CLOB
ps.setClob(2, new FileReader(new File("f:/Hello.txt")));
//2.将程序中的字符串放入CLOB
ps.setClob(2, new BufferedReader(new InputStreamReader(new ByteArrayInputStream("aaaaaaaa".getBytes()))));
            

//取出
ps = con.prepareStatement("select * from student where id=?");
ps.setObject(1, 2);
             
rs = ps.executeQuery();
while(rs.next()) {

Clob c = rs.getClob("myinfo");
Reader r = c.getCharacterStream(); //为了关闭资源，可以将r在外面声明
int temp = 0;
while((temp = r.read())!=-1) {
     System.out.print((char)temp);
   }
}
```
### BLOB

- BLOB（Binary Large Object）
  - 用于存储大量的二进制数据 
  - 大字段有些特殊，不同数据库处理的方式不一样，大字段的操作常常是以流的方式来处理的。而非一般的字段，一次即可读出数据。 
- Mysql中相关类型：
  - TINYBLOB 最大长度为$255$字节的BLOB列。
  - BLOB[(M)] 最大长度为$65K$字节的BLOB列。
  - MEDIUMBLOB 最大长度为$16M$字节的BLOB列。 
  - LONGBLOB 最大长度为$4GB$$(2^{32}–1)$字节的BLOB列。 

```java
//插入BLOB类型的数据必须使用PreparedStatement，因为BLOB类型无法使用字符串类型拼接
String sql = "UPDATE table_data SET picture=? WHERE id=?";
ps = conn.prepareStatement(sql);
ps.setBlob(1, new FileInputStream("e:/shiro.jpg"));
ps.setInt(2, 10);
ps.executeUpdate();

//取BLOB数据
String sql = "SELECT picture FROM table_data WHERE id=?";
ps = conn.prepareStatement(sql);
ps.setInt(1, 10);
rs = ps.executeQuery();
             
if(rs.next()) {
   //1.使用getBlob()方法读取到BLOB对象
   Blob picture = rs.getBlob(1);
   //2.调用BLOB的getBinaryStream()方法得到输入流，再使用IO操作即可
   InputStream in = picture.getBinaryStream();
   OutputStream out = new FileOutputStream("e:/aa.jpg");
   byte[] buffer = new byte[1024];
   int len = 0;
   while((len=in.read(buffer)) != -1) {
       out.write(buffer, 0, len);
   }
   out.flush();
   out.close();
   in.close();
}
```

# 3.事务+批处理

## 事务

- 事务基本概念     一组要么同时执行成功，要么同时执行失败的SQL语句。是数据库操作的一个执行单元！ 
- 事务开始于：
  -  连接到数据库上，并执行一条DML语句(`INSERT`、`UPDATE`或`DELETE`)。 
  - 前一个事务结束后，又输入了另外一条DML语句。 
-  事务结束于：
  - 执行`COMMIT`或`ROLLBACK`语句。
  - 执行一条DDL语句，例如`CREATE TABLE`语句；在这种情况下，会自动执行`COMMIT`语句。
  - 执行一条DCL语句，例如`GRANT`语句；在这种情况下，会自动执行`COMMIT`语句。
  -  断开与数据库的连接。
  -  执行了一条DML语句，该语句却失败了；在这种情况中，会为这个无效的DML语句执行`ROLLBACK`语句。 
- 为了让多个SQL语句作为一个事务执行：
  - 调用`Connection`对象的`setAutoCommit(false)`来取消自动提交事务。
  - 在所有的SQL语句都成功执行后，调用`commit()`方法提交。
  - 在出现异常时，调用`rollback()`方法回滚事务。
  - 若此时`Connection`没有被关闭，则需要恢复其自动提交状态 。
- 事务的四大特点（ACID） 
  - atomicity（原子性）表示一个事务内的所有操作是一个整体，要 么全部成功，要么全失败；
  - consistency（一致性）表示一个事务内有一个操作失败时，所有的更改过的数据都必须回滚到修改前的状态；
  -  isolation（隔离性） 一个事务的执行不能被其他事务干扰，即一个事务内部的操作及使用的数据对并发的其他事物是隔离的，并发执行的各个事务之间不能相互干扰。
  - durability（持久性）持久性事务完成之后，它对于系统的影响是永久性的。 

```java
/**
 * id为2的用户给id为1的用户转账500
 * 1.如果多个操作，每个操作使用的是自己单独的连接，则无法保证事务。
 */
@Test
public void testTransaction() {
    Connection conn = null;
    try {
         conn = JDBCTools.getConnection();
         //开始事务：
         conn.setAutoCommit(false);    //取消自动提交
         String sql = "UPDATE account SET balance = balance-500 WHERE id = 2";
         update(conn, sql);
         sql = "UPDATE account SET balance = balance+500 WHERE id = 1";
         update(conn, sql);
         //提交事务：
         conn.commit();
    } catch (Exception e) {
         e.printStackTrace();
         
         //回滚事务
         try {
             conn.rollback();
         } catch (SQLException e1) {
             e1.printStackTrace();
         }
    } finally {
         JDBCTools.release(null, null, conn);
    }
}
public static void update(Connection conn, String sql, Object... args) {
    PreparedStatement ps = null;
    try {
         ps = conn.prepareStatement(sql);
         for (int i = 0; i < args.length; i++) {
             ps.setObject(i + 1, args[i]);
         }
         ps.executeUpdate();
    } catch (Exception e) {
         e.printStackTrace();
    } finally {
         JDBCTools.release(null, ps, null);
    }
}
```

## 隔离级别

对于同时运行的多个事务，当这些事务访问数据库中相同的数据时，如果没有采取必要的隔离机制，会导致各种并发问题： 

- 脏读：两个事务T1、T2，T1读取了已经被T2更新但还没有被提交的字段，之后，若T2回滚，T1读取的内容就是临时且无效的；
- 不可重复读（即 原始读取不可重复）：两个事务T1、T2，T1读取了一个字段，然后T2更新了该字段，之后，T1再次读取同一个字段，值就不同了；重点在`UPDATE`和`DELETE`；
- 幻读：两个事务T1、T2，T1从一个表中读取了一个字段，然后T2在该表中插入一些新的行，之后，如果T1再次读取同一个表就会多出几行。重点在`INSERT`。 

 脏读是要避免的。 



- 事务隔离级别从低到高 
  - 读取未提交（Read Uncommitted)：允许事务读取未被其他事务提交的变更。脏读、不可重复读和幻读都会出现。
  - 读取已提交(Read Committed)：只允许事务读取已被其他事务提交的变更，可以避免脏读，但不可重复读和幻读仍会出现。
  - 可重复读（Repeatable Read)：确保事务可以多次从一个字段中读取相同的值，在这个事务持续期间，禁止其他事务对这个字段进行更新，可以避免脏读和不可重复读，但幻读仍存在。
  - 序列化（serializable)：确保事务可以从一个表中读取相同的行，在这个事务持续期间，禁止其他事务对该表执行插入、更新和删除操作，多数并发问题都可避免，但性能十分低下。 

Oracle支持2种事务隔离级别：Read committed、serializable。默认的事务隔离级别是Read committed；

MySQL支持4种事务隔离级别，默认的事务隔离级别是Repeatable Read。 

```java
/**
 * 测试事务的隔离级别
 * 在JDBC种可以通过 Connection 的 setTransactionIsolation() 方法来设置事务的隔离级别
 */
@Test
public void testTransactionIsolationUpdate() {
  Connection conn = null;
  try {
    conn = JDBCTools.getConnection();
    conn.setAutoCommit(false);
    String sql = "UPDATE account SET balance = balance-500 WHERE id = 2";
    update(conn, sql);
    //此处设置断点，该测试方法用来debug，而read测试方法run，就可观察未提交时读到的数据
    conn.commit();    
  } catch (Exception e) {
    e.printStackTrace();
  } finally {
    JDBCTools.release(null, null, conn);   
  }
}
/**
     * 测试隔离级别
     * getForValue()方法中设置了隔离级别
     * conn.setTransactionIsolation(Connection.TRANSACTION_READ_UNCOMMITTED);此时，如果update没有提交，read也可以读取到未提交的数据
     * conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);此时，只有update提交后，read的才能读取到修改的数据，否则读取的是未提交的数据
     */
@Test
public void testTransactionIsolationRead() {
  String sql = "SELECT balance FROM account WHERE id=2";
  Integer balance = getForValue(sql);
  System.out.println(balance);
}
```

```java
//getForValue()方法中设置隔离级别
conn = JDBCTools.getConnection();
//设置隔离级别
//conn.setTransactionIsolation(Connection.TRANSACTION_READ_UNCOMMITTED);
conn.setTransactionIsolation(Connection.TRANSACTION_READ_COMMITTED);
ps = conn.prepareStatement(sql);
```

## 批处理Batch

- JDBC的批量处理语句包括下面两个方法： 
  - `stmt.addBatch()`:添加需要批量处理的SQL语句或参数；
  - `executeBatch()`:执行批量处理语句 
- 通常会遇到两种批量处理执行SQL语句的情况：
  - 多条SQL语句的批量处理；
  - 一个SQK语句的批量传参。 

```java
conn = JDBCTools.getConnection();
JDBCTools.beginTransaction(conn);
sql = "INSERT INTO account(user,password,account) VALUES(?,?,?)";
ps = conn.prepareStatement(sql);
for (int i = 0; i < 1000000; i++) {
  ps.setString(1, "Nan" + i);
  ps.setString(2, "132128" + i + "fS");
  ps.setInt(3, 1000 + i);
  // 积攒SQL
  ps.addBatch();
  // 当积攒到一定程度，就统一的执行一次，并清空先前积攒的SQL
  if ((i + 1) % 300 == 0) {
    ps.executeBatch();
    ps.clearBatch();
  }
}
//如果总条数不是批量数值的整数倍，则最后还需要执行一次
if(1000000%300!=0) {
  ps.executeBatch();
  ps.clearBatch();
}

＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊＊
Connection con = null;
Statement stmt = null;      //使用Statemrnt
try {
  //创建连接。。。

  con.setAutoCommit(false);//设置为手动提交
  stmt = con.createStatement();

  for(int i=0;i<20000;i++) {        //增加20000行数据
    stmt.addBatch("insert into student(id,uname,age) values("+i+",'Nan"+i+"',18)");

    //使用Batch
  }
  stmt.executeBatch(); //使用Batch

  con.commit();//提交
```

# 4.数据库连接池

## 介绍

数据库连接池负责分配、管理和释放数据库连接，它允许应用程序重复使用一个现有的数据库连接，而不是重新建立一个。

优点：资源重用、更快的系统反应速度、新的系统分配手段、统一的连接管理，避免数据库连接泄露。

JDBC的数据库连接池使用 `javax.sql.DataSource `来表示，这是一个接口，通常由服务器（weblogic、WebSphere、Tomcat）提供实现，也有开源组织提供实现： DBCP数据库连接池；C3P0数据库连接池 

引入 jar 包：

```xml
<dependency>
  <groupId>commons-dbcp</groupId>
  <artifactId>commons-dbcp</artifactId>
  <version>1.4</version>
</dependency>
<!--或-->
<dependency>
  <groupId>com.mchange</groupId>
  <artifactId>c3p0</artifactId>
  <version>0.9.5.2</version>
</dependency>
```

- `DataSource`称为**数据源**（或连接池），包含连接池和连接池管理两部分。 
  - 注意：
    - 数据库连接池的`Connection` 对象进行`close`操作时，并不是真的进行关闭，而是把该连接归还到数据库连接池中。
    - 数据源只能被创建一次，所以放到静态代码块中。 



## DBCP数据源

- 不使用配置文件：

```java
public class DBCPPoolTest {
  /**
     * 使用DBCP数据库连接池
     * @throws SQLException
     */
  @Test
  public void testBasicDataSource() throws SQLException {
    //1.创建DBCP数据源实例
    //这里的final是为了后面的线程这一匿名内部类可以调用dataSource外部局部变量
    final BasicDataSource dataSource = new BasicDataSource();

    //2.为数据源实例指定必须的属性
    dataSource.setUsername("root");
    dataSource.setPassword("132128");
    dataSource.setUrl("jdbc:mysql:///spring-3?serverTimezone=UTC");
    dataSource.setDriverClassName("com.mysql.cj.jdbc.Driver");//MySQL 6.0后可以不写

    //3.为数据源实例指定可选的属性
    //1)指定数据库连接池中初始化连接的个数
    dataSource.setInitialSize(5);
    //2)指定最大连接数：同一时间可以同时向数据库申请的连接数
    dataSource.setMaxActive(5);
    //3)指定最小连接数：在数据库连接池中保存的最少的空闲连接数
    dataSource.setMinIdle(5);
    //4)等待数据库连接池分配连接的最长时间，单位为毫秒，超出时间会抛异常
    dataSource.setMaxWait(1000 * 5);

    //4.从数据源中获取数据库连接
    Connection conn = dataSource.getConnection();
    System.out.println(conn.getClass());

    conn = dataSource.getConnection();
    System.out.println(conn.getClass());

    conn = dataSource.getConnection();
    System.out.println(conn.getClass());

    conn = dataSource.getConnection();
    System.out.println(conn.getClass());

    Connection conn2 = dataSource.getConnection();
    System.out.println("第五个>" + conn2.getClass());

    //下面的操作和上面设置的可选属性有关
    //获取第六个连接时，由于前5个连接没有被释放，等待5秒后抛出异常
    //       conn = dataSource.getConnection();
    //       System.out.println(conn.getClass());


    //3秒后释放一个Connection，然后使用线程再获取第六个连接
    new Thread(() -> {
      Connection con = null;
      try {
        con = dataSource.getConnection();
        System.out.println(con.getClass());
      } catch (SQLException e) {
        e.printStackTrace();
      }
    }).start();


    try {
      Thread.sleep(3000);    //休眠时间超过上面设置得等待时间会抛出异常
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    //释放connection
    conn2.close();
  }
```

- 配置文件 dbcp.properties

```properties
username=root
password=123456
url=jdbc:mysql:///spring-3?serverTimezone=UTC
driverclassname=com.mysql.cj.jdbc.Driver

initialSize=10  #指定数据库连接池中初始化连接的个数
maxActive=50  #指定最大连接数：同一时间可以同时向数据库申请的连接数
minIdle=5  #指定最小连接数：在数据库连接池中保存的最少的空闲连接数
maxWait=5000  #等待数据库连接池分配连接的最长时间，单位为毫秒，超出时间会抛异常
```

```java
public class DBCPPoolTest {  
  /** 使用BasicDataSourceFactory*/
  @Test
  public void testDataSourceFactory() throws Exception {
    //1.加载dbcp的properties配置文件：配置文件中的键需要来自BasicDataSource的属性
    Properties properties = new Properties();
    Properties.load(DBCPPoolTest.class.getClassLoader().
                    getResourceAsStream("dbcp.properties"));

    //2.调用BasicDataSourceFactory的createDataSource()方法创建DataSource实例
    DataSource dataSource = BasicDataSourceFactory.createDataSource(properties);
    //3.从DataSource势力中获取数据库连接
    System.out.println(dataSource.getConnection());

    //测试配置文件配置的属性是否成功
    BasicDataSource basicDataSource = (BasicDataSource) dataSource;
    System.out.println(basicDataSource.getMaxWait());
  }
}
```

## C3P0数据源

C3P0帮助文档：<https://www.mchange.com/projects/c3p0/> 

配置文件 c3p0-config.xml 

```xml
<?xml version="1.0" encoding="UTF-8"?>
<c3p0-config>
  <named-config name="helloC3P0">
    <!-- 指定连接数据源的基本属性 -->
    <property name="user">root</property>
    <property name="password">132128</property>
    <property name="jdbcUrl">jdbc:mysql:///spring-3?serverTimezone=UTC</property>
    <property name="driverClass">com.mysql.cj.jdbc.Driver</property>

    <!-- 若数据库中连接数不足时，一次向数据库服务器申请的连接数 -->
    <property name="acquireIncrement">5</property>
    <!-- 初始化数据库连接池时的连接数 -->
    <property name="initialPoolSize">5</property>
    <!-- 数据库连接池中最小的连接数 -->
    <property name="minPoolSize">5</property>
    <!-- 数据库连接池中最大的连接数 -->
    <property name="maxPoolSize">10</property>
    <!-- C3P0数据库连接池可以维护的Statement的个数 -->
    <property name="maxStatements">20</property>
    <!-- 每个连接可以同时使用的Statement的个数 -->
    <property name="maxStatementsPerConnection">5</property>
  </named-config>
</c3p0-config>
```

```java
public class C3P0PoolTest {
  @Test //手动配置
  public void testC3P0() throws PropertyVetoException, SQLException {
    ComboPooledDataSource cpds = new ComboPooledDataSource();
    cpds.setDriverClass("com.mysql.cj.jdbc.Driver");
    cpds.setJdbcUrl("jdbc:mysql:///spring-3?serverTimezone=UTC");
    cpds.setUser("root");
    cpds.setPassword("132128");

    System.out.println(cpds.getConnection());
  }
  /**
     * 1.创建 c3p0-config.xml，参考帮助文档中的  Appendix B: Configuation Files, etc.
     * 2.创建 ComboPooledDataSource("helloC3P0") 实例，其中参数为配置文件中的<named-config>的name属性值
     * 3.从 DataSource 中获取数据库连接
     * @throws SQLException
     */
  @Test
  public void testC3P0WithConfigFile() throws SQLException {
    DataSource dataSource = new ComboPooledDataSource("helloC3P0");
    System.out.println(dataSource.getConnection());

    ComboPooledDataSource cpds = (ComboPooledDataSource) dataSource;
    System.out.println(cpds.getMaxPoolSize());
    System.out.println(cpds.getMaxStatements());

  }
}
```

# 5.DBUtils 工具类

​	 DBUtils 是 Apache 提供的一个开源JDBC工具类。 可以借助该工具类来写方法

```xml
<dependency>
  <groupId>commons-dbutils</groupId>
  <artifactId>commons-dbutils</artifactId>
  <version>1.7</version>
</dependency>
```



## 利用工具类

- 创建 `QueryRunner` 实例 ，DBUtils中的`QueryRunner`类是线程安全的，所以可以放到方法外面，作为全局变量 

- 更新方法

  ```java
  /**
   * 测试 QueryRunner 类的 Update 方法
   * 该方法可用于INSERT,UPDATE和DELETE
   * @throws SQLException
   */
  @Test
  public void testQueryRunnerUpdate() throws SQLException {
    //1.创建 QueryRunner 实例
    QueryRunner qr = new QueryRunner();
    String sql = "DELETE FROM table_data WHERE id IN (?,?)";
    Connection conn = null;
    try {
      conn = JDBCTools.getConnection();
      //2.调用update方法
      qr.update(conn, sql, 8, 9);
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      JDBCTools.release(null, null, conn);
    }
  }
  ```

- 查询方法

  ```java
  /**
       * QueryRunner 的Query()方法的返回值取决于传入的 ResultSetHandler 参数的 handle() 方法的返回值
       */
  @Test
  public void testQuery() {
  
    QueryRunner qr = new QueryRunner();
    Connection conn = null;
    try {
      conn = JDBCTools.getConnection();
      String sql = "SELECT id,last_name,email,depi_id FROM table_data WHERE ID>3 ";
      List<Student> list = qr.query(conn, sql, new MyResultSetHandler());
      System.out.println(list);
    } catch (Exception e) {
      e.printStackTrace();
    } finally {
      JDBCTools.release(null, null, conn);
    }
  }
  class MyResultSetHandler implements ResultSetHandler<List<Student>>{ //这是一个内部类
    /**
            * handle()方法的返回值会作为Query()方法的返回值返回
            */
    @Override
    public List<Student> handle(ResultSet rs) throws SQLException {
      List<Student> list = new ArrayList<>();
      while(rs.next()) {
        Integer id = rs.getInt(1);
        String last_name = rs.getString(2);
        String email = rs.getString(3);
        int depi_id = rs.getInt(4);
        Student student = new Student(id, last_name, email, depi_id);
        list.add(student);
      }  
      return list;
    }
  }
  ```



## DBUtils中的Handler

- `BeanHandler`：

  - 把结果集的第一条记录转为创建`BeanHandler`对象时传入的`Class`参数对应的对象 

    ```java
    @Test
    public void testBeanHandler() {
      Connection conn = null;
      try {
        conn = JDBCTools.getConnection();
        String sql = "SELECT id,last_name,email,depi_id FROM table_data WHERE ID=? ";
        Student student = qr.query(conn, sql, new BeanHandler<>(Student.class), 7);
        System.out.println(student);
      } catch (Exception e) {
        e.printStackTrace();
      } finally {
        JDBCTools.release(null, null, conn);
      }
    }
    ```

- `BeanListHandler`

  - 把结果集转为一个`List`，该`List`不为`null`，但可能为空集合（即`size()`返回值为0）

  - 若SQL语句的确能查询到记录，`List`中存放创建`BeanListHandler`传入的`Class`参数对应的对象 

    ```java
    @Test
    public void testBeanListHandler() {
      //。。。
      String sql = "SELECT id,last_name,email,depi_id FROM table_data WHERE ID>? ";
      List<Student> students = qr.query(conn, sql, new BeanListHandler<>(Student.class), 2);
      System.out.println(students);
      //。。。同上一个方法
    }
    ```

- `MapHandler`

  - 返回SQL对应的第一条记录对应的`Map`对象

  - 键：SQL查询的列名（不是别名），值：列的值 

    ```java
    @Test
    public void testMapHandler() {
      //。。。
      String sql = "SELECT id,last_name,email,depi_id FROM table_data WHERE ID>? ";
      Map<String,Object> result = qr.query(conn, sql, new MapHandler(), 2);
      System.out.println(result);
      //。。。
    }
    ```

- `MapListHandler`

  - 返回多条记录对应的`Map`的集合将结果集转为一个`Map`的`List`

  - `Map`对应查询的一条记录。键：SQL查询的列名（不是别名），值：列的值 

    ```java
    @Test
    public void testMapListHandler() {
      //。。。
      String sql = "SELECT id,last_name,email,depi_id FROM table_data WHERE ID>? ";
      List<Map<String,Object>> results = qr.query(conn, sql, new MapListHandler(), 2);
      System.out.println(results);
      //。。。
    }
    ```

- `ScalarHandler`把结果集转为一个数值（可以是任意基本数据类型和字符串、`Date`等）返回

  - 如果有多列，默认返回第一列的值 

    ```java
    @Test
    public void testScalarHandler() {
      //。。。
      String sql = "SELECT last_name FROM table_data WHERE ID=? ";
      String result = qr.query(conn, sql, new ScalarHandler<>(), 2);
      System.out.println(result);
      //。。。
    }
    ```



# 6.存储过程

- 调用存储过程步骤：
  - 通过`Connection` 对象的 `prepareCall（）`方法创建一个 `CallableStatement `对象的实例。再使用 `prepareCall（）`方法时，需要传入一个 String 类型的字符串，该字符串用于指明如何调用存储过程；
  - 通过 `CallableStatement `对象的` registerOutParameter（）` 方法**注册OUT参数**；
  - 通过 `CallableStatement` 对象的 `setXxx（）`方法设定 **IN 或 IN　OUT参数**，若想将参数值设为null，可以使用`setNull（）`方法；
  - 通过` CallableStatement` 对象的 `execute（）`方法执行存储过程；
  - 如果所调用的是带返回参数的存储过程，还需要通过 `CallableStatement` 对象的` getXxx（）`方法获取返回值。 