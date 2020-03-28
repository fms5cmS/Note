# JPA

Java Persistence API：用于对象持久化的API。Java EE5.0 平台标准的 ORM 规范，使得应用程序以统一的方式访问持久层。

- JPA的优势：
  - 标准化：:  提供相同的API，保证基于JPA开发的企业应用能够经过少量的修改就能够在不同的JPA框架下运行；
  - 简单易用，集成方便：JPA 的主要目标之一就是提供更加简单的编程模型，在JPA 框架下创建实体和创建Java  类一样简单，只需要使用`javax.persistence.Entity`进行注释；JPA的框架和接口也都非常简单；
  - 可媲美JDBC的查询能力：JPA的查询语言是面向对象的，JPA定义了独特的JPQL，而且能够支持批量更新和修改、JOIN、GROUPBY、HAVING等通常只有SQL 才能够提供的高级查询特性，甚至还能够支持子查询；
  - 支持面向对象的高级特性：JPA 中能够支持面向对象的高级特性，如类之间的继承、多态和类之间的复杂关系，最大限度的使用面向对象的模型



## JPA基本注解

- `@Entity`：标注用于实体类声明语句之前，**指出该Java类为实体类，将映射到指定的数据库表**。
  - 如声明一个实体类`Customer`，它将映射到数据库中的customer表上。
  - 实体类必须有一个public或protected的无参构造器
  - 实体实例被当作值以分离对象方式进行传递（如通过会话bean的远程业务接口），则该类必须实现`Serializable`接口
- `@Table`：当**实体类与其映射的数据库表名不同名时**需要使用`@Table`标注说明，该标注与`@Entity`标注并列使用，置于实体类声明语句之前，可写于单独语句行，也可与声明语句同行
  - name属性用于指明数据库的表名
  - catalog和schema属性设置表所属的数据库目录或模式，通常为数据库名
  - uniqueConstraints属性用于设置约束条件，通常不须设置
- `@Id`：**声明一个实体类的属性映射为数据库的主键列**。该属性通常置于属性声明语句之前，可与声明语句同行，也可写在单独行上，也可标注在属性的getter方法前
- `@GeneratedValue`：标注主键的生成策略，通过 strategy属性指定。默认情况下，JPA自动选择一个最适合底层数据库的主键生成策略：SqlServer对应identity，MySQL对应auto_increment。有以下几种策略：
  - `javax.persistence.GenerationType.IDENTITY`：用数据库ID自增长的方式来自增主键字段，Oracle
    不支持；
  - `javax.persistence.GenerationType.AUTO`：JPA自动选择合适的策略，是默认选项
  - `javax.persistence.GenerationType.SEQUENCE`：通过序列产生主键，通过`@SequenceGenerator`注解指定序列名，MySql不支持；
  - `javax.persistence.GenerationType.TABLE`：通过表产生主键，框架借由表模拟序列产生主键，使用该策略可以使应用更易于数据库移植
- `@Basic`：表示一个简单的属性到数据库表的字段的映射,对于没有任何标注的 getXxxx() 方法,默认即为`@Basic`
  - fetch: 表示该属性的读取策略,有EAGER 和 LAZY两种,分别表示主支抓取和延迟加载,默认为EAGER；
  - optional:表示该属性是否允许为null，默认为true
- `@Column`：**当实体的属性与其映射的数据库表的列不同名时**需要使用`@Column`标注说明，该属性通常置于实体的属性声明语句之前，还可与`@Id`标注一起使用。
  - name属性用于设置映射数据库表的列名
  - columnDefinition属性表示该字段在数据库中的实际类型
    - `Date`类型仍无法确定数据库中字段类型究竟是`DATE`、`TIME`还是`TIMESTAMP`。此外,`String`的默认映射类型为`VARCHAR`，如果要将`String`类型映射到特定数据库的BLOB或TEXT字段类型。
- `@Transient`：**表示该属性并非一个到数据库表的字段的映射,ORM框架将忽略该属性**。
  - **如果一个属性并非数据库表的字段映射,就务必将其标示为`@Transient`,否则,ORM框架默认其注解为`@Basic`。**
- `@Temporal`：在进行属性映射时可使用`@Temporal`注解来调整精度（JavaAPI中没有定义`Date`的精度，而数据库中则有`DATE`、`TIME`和`TIMESTAMP`三种）

- 关系：
  - 一对一：`@OneToOne`
  - 一对多：`@OneToMany`
  - 多对一：`@ManyToOne`
  - 多对多：`@ManyTOMany`



## JPA API

- `Persistence`类用于获取`EntityManagerFactory`实例。包含一个名为`createEntityManagerFactory()`的静态方法 。
- `EntityManagerFactory`：用来创建`EntityManager`实例：
  - `createEntityManager()`用于创建实体管理器对象实例
  - `createEntityManager(Map map)`用于创建实体管理器对象实例的重载方法，`Map`参数用于提供`EntityManager`的属性
  - `isOpen()`检查`EntityManagerFactory`是否处于打开状态。实体管理器工厂创建后一直处于打开状态，除非调用`close()`方法将其关闭。
  - `close()`关闭`EntityManagerFactory`。`EntityManagerFactory`关闭后将释放所有资源，`isOpen()`方法测试将返回false，其它方法将不能调用，否则将导致`IllegalStateException`异常
- `EntityManager`接口
  - 定义用于与持久性上下文交互的方法
  - 创建、删除持久实体实例，通过实体的主键查找实体
  - 运行在实体上运行查询



# Spring Data

Spring Data是Spring 的一个子项目。用于简化数据库访问，**支持NoSQL 和关系数据存储**。

- 支持的NoSQL存储：
  - MongoDB （文档数据库）
  - Neo4j（图形数据库）
  - Redis（键/值存储）
  - Hbase（列族数据库）
- 支持的关系数据存储：
  - JDBC
  - **JPA**

Spring Data为我们提供使用统一的API来对数据访问层进行操作；这主要是Spring Data Commons项目来实现的。Spring Data Commons让我们在使用关系型或者非关系型数据访问技术时都基于Spring提供的统一标准，标准包含了CRUD（创建、获取、更新、删除）、查询、排序和分页的相关操作。在配置类上使用`@EnableJpaRepository`，就可以自动发现有`Repository`接口的扩展。

- 统一的`Repository`接口
  - `Repository<T, ID>`：统一接口
    - `RevisionRepository<T, ID, N extends Number & Comparable<N>>`：基于乐观锁机制
    - `CrudRepository<T, ID>`：基本CRUD操作
    - `PagingAndSortingRepository<T, ID>`：基本CRUD及分页
      - 会使用到排序的`Sort`类、记录分页信息的`Pageable`类

提供数据访问模板类xxxTemplate：如：`RedisTemplate`、`MongoTemplate`。。。

- 根据方法名定义查询：
  - find...By..  或  read...By...  或  query...By...  或 get...By...
  - count...By...
  - ...OrderBy...[Asc 或 Desc]
  - And 或 Or 或 IgnoreCase
  - Top 或 First 或 Distinct



## Spring Data JPA

Spring Data JPA ：致力于减少数据访问层(DAO)的开发量。开发者唯一要做的，就只是**声明持久层的接口**，不需要编写接口实现类，其他都交给Spring Data JPA 来帮你完成！底层默认依赖 Hibernate JPA实现。

- `JpaRepository`基本功能：编写接口继承`JpaRepository`既有crud及分页等基本功能
- 在接口中只需要声明符合规范的方法，即拥有对应的功能
- `@Query`自定义查询，定制查询SQL
- Specifications查询（Spring Data JPA支持JPA2.0的Criteria查询）



### 使用

导入包：

```xml
<dependency>
    <groupId>org.springframework.data</groupId>
    <artifactId>spring-data-jpa</artifactId>
    <version>2.1.2.RELEASE</version>
</dependency>
```

使用 SpringData JPA 进行持久层开发需要的四个步骤：

1. ==配置Spring整合JPA==；

2. ==在 Spring配置文件中配置 Spring Data==，让 Spring为声明的接口创建代理对象。配置了\<jpa:repositories>
   后，Spring初始化容器时会扫描base-package  指定的包目录及其子目录，为继承`Repository`或其子接口的接口创建代理对象，并将代理对象注册为SpringBean，业务层便可以通过 Spring自动封装的特性来直接使用该对象。

   ```xml
   <!--扫描dao接口所在包-->
   <jpa:component-scan base-package="com.zzk.dao"/>
   ```

3. ==声明持久层的接口==，该Dao接口继承`Repository`，`Repository`是一个标记型接口，它不包含任何方法，如必要，Spring Data 可实现`Repository`其他子接口，其中定义了一些常用的增删改查，以及分页相关的方法。

   ```java
   public interface UserDao extends JpaRepository<User,Integer> {  }
   ```

4. ==在接口中声明需要的方法==。SpringData 将根据给定的策略来为其生成实现代码。



### SpringBoot整合

- 引入 spring-boot-starter-data-jpa

  ```xml
  <dependency>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-data-jpa</artifactId>
  </dependency>
  ```

- 修改application.yml：

  ```yaml
  spring:
    datasource:
      username: root
      password: 132128
      driver-class-name: com.mysql.cj.jdbc.Driver
      url: jdbc:mysql://localhost:3306/myemployees?serverTimezone=UTC&useSSL=false
    jpa:
      hibernate:
        # update：如果启动应用后没有表则在数据库中创建，如果已有且表结构发生改变则更新数据库中的表
        # create-drop：每次运行程序后会创建表结构，运行结束后会删除表结构
        ddl-auto: update
      #在控制台显示每次的SQL
      show-sql: true
  ```

- 编写一个实体类(bean)和数据表进行映射，并且配置好映射关系

  ```java
  @Entity  //告诉JPA这是一个实体类（和数据表映射的类）
  @Table(name = "author") //指定和哪个数据表对应;如果省略默认表名就是author
  public class Author {
      @Id   //这是主键
      @GeneratedValue(strategy = GenerationType.IDENTITY) //自增主键
      private Integer id;
      @Column(name = "au_name",length = 20) //这是和数据表对应的一个列
      private String au_name;
      @Column  //默认对应数据表列名就是属性名
      private String nation;
      //注意，无参构造器要设置为protected，这是JPA规范要求，防止直接使用！！！！！
  	//getter、setter方法
  }
  ```

- 编写一个Dao接口来操作实体类对应的数据表（Repository）

  ```java
  public interface UserRepository extends JpaRepository<Author,Integer> { }
  ```

- 使用时注入Dao接口，然后使用它就可以了

  ```java
  @RestController
  public class UserController {
      @Autowired
      private UserRepository userRepository;
  
      @GetMapping("/user")
      public Author insertAuthor(Author author){
          return userRepository.save(author);
      }
  }
  ```

  









