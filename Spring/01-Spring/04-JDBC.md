JDBC Template是Spring提供的简化数据库操作的工具。

# 4.1 部署（C3P0）

```xml
<!-- 导入数据源文件 -->
<context:property-placeholder location="classpath:db.properties"/>
<!-- 配置C3P0数据源 -->
<bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
    <property name="user" value="${jdbc.user}"></property>
    <property name="password" value="${jdbc.password}"></property>
    <property name="driverClass" value="${jdbc.driverClass}"></property>
    <property name="jdbcUrl" value="${jdbc.url}"></property>
    <property name="initialPoolSize" value="${jdbc.initPoolSize}"></property>
    <property name="maxPoolSize" value="${jdbc.maxPoolSize}"></property>
</bean>
<!-- 配置Spring的JDBCTemplate -->
<bean id="jdbcTemplate" class="org.springframework.jdbc.core.JdbcTemplate">
    <property name="dataSource" ref="dataSource"></property>
</bean>
```

# 4.2 创建方法

```java
    private ApplicationContext ctx = null;
    private JdbcTemplate jdbcTemplate = null;
    {
         ctx = new ClassPathXmlApplicationContext("ApplicationContext.xml");
         jdbcTemplate = (JdbcTemplate) ctx.getBean("jdbcTemplate");
    }
```

创建CRUD方法：

- 用 sql 语句和参数更新数据库：update() ：

```java
/**执行INSERT,UPDATE,DELETE*/
    @Test
    public void testUpdate() {
         String sql = "INSERT INTO table_data VALUES(?,?,?,?,?)";
         jdbcTemplate.update(sql, 6,"Bx","bx@qq.com",5,null);
    }
```

- 批量更新数据库：batchUpdate() :

```java
	/**
     * 执行批量更新
     * 一条记录的可变参数是一个Object[]数组，多条记录是一个Object[]数组的List
     */
    @Test
    public void testBatchUpdate() {
         String sql = "INSERT INTO table_data(last_name,email,depi_id) VALUES(?,?,?)";
         List<Object[]> list = new ArrayList<>();
         list.add(new Object[] {"AA","aa@123.com",1});
         list.add(new Object[] {"bb","BB@123.com",2});
         list.add(new Object[] {"CC","CC@123.com",4});
         list.add(new Object[] {"DD","DD@123.com",3});
         jdbcTemplate.batchUpdate(sql, list);
    }
```

- 查询单行：queryForObject() 

```java
	/**
     * 从数据库获取一条记录，实际得到一个对应的对象
     * 注意：不是调用queryForObject(String sql, Class<T> requiredType, Object... args)方法
     * 而是需要调用queryForObject(String sql, RowMapper<Student> rowMapper, Object... args) 方法
     * 1.其中 RowMapper 指定如何去映射结果集所在的行，常用的实现类为BeanPropertyRowMapper
     * 2.使用SQL中列的别名完成列名和类的属性名的映射，如：last_name name
     * 3.不支持级联属性。JdbcTemplate只是JDBC的一个小工具，而不是ORM框架
     */
    @Test
    public void testQueryForObject() {
         String sql = "SELECT id,last_name name,email FROM table_data WHERE id=?";
         RowMapper<Student> rowMapper = new BeanPropertyRowMapper<>(Student.class);
         Student student = jdbcTemplate.queryForObject(sql, rowMapper,7);
         System.out.println(student);
    }
```

- 查询多行：query() :

```java
    /**
     * 得到实体类的集合
     * 注意：调用的不是QueryForList()
     */
    @Test
    public void testQueryForList() {
         String sql = "SELECT id,last_name name,email FROM table_data WHERE id>?";
         RowMapper<Student> rowMapper = new BeanPropertyRowMapper<>(Student.class);
         List<Student> students = jdbcTemplate.query(sql, rowMapper,5);
         System.out.println(students);
    }
```

- 单值查询：queryForObject() 

```java
    /**
     * 获取单个列的值，或做统计查询
     * 使用queryForObject(String sql, Class<Long> requiredType) 方法
     */
    @Test
    public void testQueryForObject2() {
         String sql = "SELECT count(id) FROM table_data";
         long count = jdbcTemplate.queryForObject(sql, Long.class);
         System.out.println(count);
    }
```



# 4.3 DAO

JdbcTemplate类被设计成线程安全的，所以可以在IOC容器中声明它的单个实例，并将这个实例注入到所有的DAO实例中。    推荐直接使用 JdbcTemplate作为DAO类的成员变量。eg：

```java
@Repository
public class StudentDAO {
    @Autowired
    private JdbcTemplate jdbcTemplate;
    
    public Student get(int id) {
         String sql = "SELECT id,last_name name,email FROM table_data WHERE id=?";
         RowMapper<Student> rowMapper = new BeanPropertyRowMapper<>(Student.class);
         Student student = jdbcTemplate.queryForObject(sql, rowMapper,id);
         return student;
    }
}
```

Spring JDBC框架还提供了一个JdbcDaoSupport类来简化DAO实现，该类声明了 JdbcTemplate属性，它可以从IoC容器中注入，或者自动从数据源中创建。**不推荐**



# 4.4 具名参数

经典的JDBC用法中，SQL参数是用占位符 ？表示，且受到位置限制，定位参数的问题在于，一旦参数顺序变化，就必须改变参数绑定；Spring JDBC框架中，绑定SQL参数的另一种选择是使用具名参数（named parameter）：

具名参数：SQL按名称（以冒号开头）而不是按位置进行指定，具名参数更易维护，也提升了可读性。具名参数由框架类在运行时用占位符取代。具名参数只在`NamedParameterJdbcTemplate`中得到支持 ！

```java
/**
   * 可以为参数起名字
   * 1.好处：若有多个参数，则不用再去对应位置，直接对应参数名
   * 2.缺点：较为麻烦
   */
    @Test
    public void testNamedParameterJdbcTemplate() {
         String sql = "INSERT INTO table_data(last_name,email,depi_id) VALUES(:ln,:email,:deptid)";
         Map<String, Object> map = new HashMap<>();
         map.put("ln", "FF");
         map.put("email", "FF@qq.com");
         map.put("deptid", 3);
         namedParameterJdbcTemplate.update(sql, map);
    }

/**
   * 使用具名参数时，可以使用 update(String sql, SqlParameterSource paramSource) 方法进行更新操作
   * 1.SQL语句中的参数名和类的属性名一致！
   * 2.使用 SqlParameterSource 的 BeanPropertySqlParameterSource 实现类作为参数
   */
    @Test
    public void testNamedParameterJdbcTemplate2() {
         String sql = "INSERT INTO table_data(last_name,email,depi_id) VALUES(:name,:email,:departId)";
         Student student = new Student();
         student.setName("xyz");
         student.setEmail("xyz@sina.com");
         student.setDepartId(5);
         
         SqlParameterSource paramSource = new BeanPropertySqlParameterSource(student);
         namedParameterJdbcTemplate.update(sql, paramSource);
    }
```

