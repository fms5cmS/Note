# 5.1 认识

MyBatis 包含一个强大的查询缓存特性，它可以非常方便地配置和定制。缓存可以极大的提升查询效率。

- MyBatis 默认定义了两级缓存： 一级缓存和二级缓存：
  - 一级缓存（本地缓存） 
    - 与数据库同一次会话期间查询到的数据放到本地缓存中，如果以后有相同的数据从缓存中拿，不查询数据库
  - 二级缓存（全局缓存） 



1. 默认情况下，只有一级缓存（`SqlSession`级别的缓存，也称为本地缓存）开启； 
2. 二级缓存需要手动开启和配置，是基于`namespace` 级别的缓存；
3. 为了提高扩展性，MyBatis定义了缓存接口 `Cache`。可以通过实现该接口来自定义二级缓存。

- 缓存的使用顺序： 
  - 新的会话开始：先看二级缓存是否有数据——>再看一级缓存中是否有数据——>如果都没有，才会通过数据库进行查询 



# 5.2 一级缓存

一级缓存（本地缓存）**`sqlSession` 级别的缓存，默认一直开启，无法关闭** 。每个sqlSession拥有单独的一级缓存。

```java
EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
Employee emp01 = mapper.getEmpById(3);

Employee emp02 = mapper.getEmpById(3);
System.out.println(emp01==emp02);                    //true。且只会发出一次sql查询请求
```



- **一级缓存失效情况**（没有使用到当前一级缓存的情况，效果就是，还需要再向数据库发出查询）

  1. ` sqlSession`不同；

     ```java
     SqlSession sqlSession = sqlSessionFactory.openSession();      //第一个 SqlSession
     EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
     Employee emp01 = mapper.getEmpById(3);
     
     SqlSession sqlSession1 = sqlSessionFactory.openSession();     //第二个 SqlSession
     EmployeeMapper mapper1 = sqlSession1.getMapper(EmployeeMapper.class);
     Employee emp02 = mapper1.getEmpById(3);
     System.out.println(emp01==emp02);                 //false。且会发出两次sql请求查询
     ```

  2. `sqlSession`相同，查询条件不同（因为当前缓存中还没有这个数据） ；

  3. `sqlSession`相同，两次查询之间执行了增删改查操作（操作对当前数据有影响） ；

  4. `sqlSession`相同，手动清除一级缓存  `openSession.clearCache()`。

     ```java
     EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
     Employee emp01 = mapper.getEmpById(3);
     sqlSession.clearCache();        //清除缓存
     Employee emp02 = mapper.getEmpById(3);
     System.out.println(emp01==emp02);
     ```



# 5.3 二级缓存

二级缓存（全局缓存）：基于`namespace`级别的缓存；一个`namespace`对应一个二级缓存。

- 工作机制
  - 一个会话，查询一条数据，这个数据就会被放在当前会话的一级缓存中；
  - 如果会话关闭，一级缓存中的数据会被保存到二级缓存中，新的会话查询信息就可以参照二级缓存中的内容；
  - `sqlSession`有： 
    - ①`EmployeeMapper` ==> `Employee`
    - ②`Departmentmapper` ==> `Department`
  - 不同namespace查出的数据，会放在自己的对应缓存中(MyBatis中是一个map) 



使用步骤：

1. 开启全局二级缓存配置：`cacheEnabled`默认开启，但为了防止版本更替，显式指定；
2. 在 mapper.xml 中配置`<cache></cache>`。在哪个mapper中写，哪个才会有二级缓存：
   - eviction属性:缓存回收策略（超了后怎么处理），取值：(默认`LRU`) 
     - LRU（最近最少使用）：移除最长时间不被使用的对象。
     - FIFO（先进先出）：按对象进入缓存的顺序来移除。
     - SOFT（软引用）：移除基于垃圾回收器状态和软引用规则的对象。
     - WEAK （弱引用）：更积极地移除基于垃圾收集器状态和弱引用规则地对象。
   - flushInterval属性：缓存刷新间隔，多长时间清空一次，默认不清空，设置毫秒值。
     - 默认情况是不设置，也就是没有刷新间隔，缓存仅仅调用语句时刷新。
   - readOnly属性：缓存是否只读，默认false。
     - true：只读缓存（不安全但速度快）：MyBatis 认为所有从缓存中获取数据的操作，都是只读操作，不会修改数据，为了加快获取速度，直接回将数据在缓存中的引用交给用户。 
     - false：读写缓存（速度慢但安全）：MyBatis 觉得获取的数据可能被修改，MyBatis 会利用序列化 与 反序列化克隆数据。
   - size属性：引用数目，正整数。缓存存放多少元素。
     - 代表缓存最多可以存储多少个对象，太大容易导致内存溢出。
   - type属性:指定自定义缓存的全类名。 
     - 实现cache接口 全类名放到type上。 
3. POJO 需要实现序列化接口

示例：

1. mybatis-config.xml 文件中 

   ```xml
   <settings>
       <!--开启驼峰命名法-->
       <setting name="mapUnderscoreToCamelCase" value="true"/>
       <!--显式地指定每个我们需要更改的配置值，即使它是默认的。防止版本更新带来的问题-->
       <!--开启懒加载，使得查询结果中关联的值可以在使用时再加载-->
       <setting name="lazyLoadingEnabled" value="true"/>
       <!--开启后每个属性都会直接全部加载出来，禁用的话就会按需加载-->
       <setting name="aggressiveLazyLoading" value="false"/>
       <!--开启全局二级缓存配置-->
       <setting name="cacheEnabled" value="true"/>
   </settings>
   ```

2. EmployeeMapper.xml 文件中 

   ```xml
   <?xml version="1.0" encoding="UTF-8"?>
   <!DOCTYPE mapper
   PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
   "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
   <mapper namespace="dao.EmployeeMapper">
       <cache eviction="FIFO" flushInterval="60000" ></cache>
       <select id="getEmpById" resultType="bean.Employee">
           select * from tb1_employee
           where id=#{id}
       </select>
   </mapper>
   ```

3. Employee 类中实现序列化接口 

   ```java
   public class Employee implements Serializable {。。。}
   ```

4. 测试

   ```java
   public void secondLevelCache() throws IOException {
       SqlSessionFactory sqlSessionFactory = getSqlSessionFactory();
       SqlSession sqlSession = sqlSessionFactory.openSession();
       SqlSession sqlSession2 = sqlSessionFactory.openSession();
       try {
           EmployeeMapper mapper = sqlSession.getMapper(EmployeeMapper.class);
           EmployeeMapper mapper2 = sqlSession2.getMapper(EmployeeMapper.class);
           Employee emp01 = mapper.getEmpById(3);
           System.out.println(emp01);
           sqlSession.close();    //数据默认在一级缓存中，只有会话提交或关闭以后，一级缓存中地数据才会转移到二级缓存中
           //第二次查询是从二级缓存中拿到的，并没有发送新的sql
           Employee emp02 = mapper2.getEmpById(3);
           System.out.println(emp02);
           sqlSession2.close();        //关闭会话
       } finally {
           sqlSession.close();
       }
   }
   ```

   ![测试结果](../../images/MyBatis-cache.png)

**注意**：查出的数据都会默认在一级缓存中，只有会话提交关闭以后，一级缓存中的数据才会转移到二级缓存。就是说：如果两个 session 一起关闭，就会两条语句。



# 5.4 相关设置

1. 全局缓存配置 `cacheEnabled = false`：关闭缓存（二级缓存关闭）（一级缓存一直可用）；
2. 使用`<select>`标签时有默认属性`useCache="true"`：若值为false是不使用缓存（二级缓存关闭）（一级缓存一直可用）；
3. 属性`flushCache="true"`：增删改完成后就会清除缓存（一级、二级都会被清空）；
   1. 查询标签中默认是 false，如果修改为true，每次查询之前都会清空缓存。 
   2. 增删改标签默认是true。
4. `SqlSession`的 `clearCache() `方法：只是清除当前 session 的一级缓存。
5. 全局配置`localCacheScope=session/statement`（默认为session）：本地缓存作用域。
   1. 一级缓存session：当前会话的所有数据保存在会话缓存中。
   2. `statement`：不共享，相当于禁用一级缓存。 