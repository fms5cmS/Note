# 3.0 Lombok 使用

javac 从 Java6 开始支持 JSR-269 API 规范，只要程序实现了该 API，就能在 javac 运行时得到调用。Lombok 支持了 JSR-269 API 规范。

给 IDE 安装 Lombok 插件；注意，每个项目都要引入依赖：

```xml
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.18</version>
    <scope>provided</scope>
</dependency>
```

注意：在类需要序列化、反序列化时详细控制字段。

使用：注解都是标注在类上。常用注解：

- `@Getter`:生成 public 的 Getter 方法
  - `@Getter(AccessLevel.PROTECTED)`:生成 protected 的 Getter 方法,还可以生成其他访问权限的方法
- `@Setter`:生成 public 的 Setter 方法
  - `@Setter(AccessLevel.PROTECTED)`:生成 protected 的 Setter 方法
- `@NoArgsConstructor`:生成无参构造器;
- `@AllArgsConstructor`:生成所有参数的构造器
- `@ToString`:生成打印所有属性的 toString()
  - `@ToString(exclude = {"column1","column2"})`:生成的 toString()排除一个或多个属性，相反的，使用 of 说明该方法中只要哪些属性
  - 直接使用`@ToString`的话，`toString()`只会打印当前类的属性而不会打印父类的属性，如果还要打印父类的属性，可以使用`@ToString(callSuper = true)`
- `@EqualsAndHashCode`:重写 equals()、hashCode()。也可以使用 exclude 排除属性，使用 of 说明该方法中只要哪些属性
- `@Slf4j`:生成 Logback 日志框架的`Logger`的实例：log，可以直接调用 log
- `@Log4j`:当项目使用 Log4j 日志框架时使用
- `@Data`:包含了`@Getter`、`@Setter`、`@ToString`、`@EqualsAndHashCode`，使用所有属性来生成！



# 3.1 Spring事务

- Spring中事务的**传播属性**：当事务方法被另一个事务方法调用时, 必须指定事务应该如何传播.

  - 一个方法A运行在一个开启了事务的方法B中时，A是使用原来B的事务还是开启一个新的事务？
  - Spring使用`@Transactional`的`propagation`来设置事务的传播行为。
  - 详见Spring笔记


| 传播特性                     | 值   | 描述                                 |
| ---------------------------- | ---- | ------------------------------------ |
| PROPAGATION_REQUIRED（默认） | 0    | 当前有事务就用当前的，没有就用新的   |
| PROPAGATION_SUPPORTS         | 1    | 事务可有可无，不是必须的             |
| PROPAGATION_MANDATORY        | 2    | 当前一定要有事务，不然抛出异常       |
| PROPAGATION_REQUIRES_NEW     | 3    | 无论是否有事务，都起一个新的事务     |
| PROPAGATION_NOT_SUPPORTED    | 4    | 不支持事务，按非事务方式运行         |
| PROPAGATION_NEVER            | 5    | 不支持事务，如果有事务则抛出异常     |
| PROPAGATION_NESTED           | 6    | 当前有事务就在当前事务里再起一个事务 |

 

- 事务的隔离级别，Spring使用`@Transactional`的`isolation`属性设置。

| 隔离性（√表示会发生该问题） | 值   | 脏读 | 不可重复读 | 幻读 |
| --------------------------- | ---- | ---- | ---------- | ---- |
| ISOLATION_READ_UNCOMMITTED  | 1    | ✓    | ✓          | ✓    |
| ISOLATION_READ_COMMITTED    | 2    | ✗    | ✓          | ✓    |
| ISOLATION_REPEATABLE_READ   | 4    | ✗    | ✗          | ✓    |
| ISOLATION_SERIALIZABLE      | 8    | ✗    | ✗          | ✗    |



# 3.2 SpringMVC中文乱码

- SpringMVC如何解决POST请求的中文乱码问题？

使用`CharacterEncodingFilter`过滤器，在web.xml文件中进行配置即可(该方法只能解决POST请求的乱码问题，如果是GET请求就无效了)

```xml
<!--字符编码过滤器，一定要放在所有过滤器之前-->
<filter>
    <filter-name>characterEncodingFilter</filter-name>
    <filter-class>org.springframework.web.filter.CharacterEncodingFilter</filter-class>
    <init-param>
        <param-name>encoding</param-name>
        <param-value>utf-8</param-value>
    </init-param>
    <init-param>
        <param-name>forceRequestEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
    <init-param>
        <param-name>forceResponseEncoding</param-name>
        <param-value>true</param-value>
    </init-param>
</filter>
<filter-mapping>
    <filter-name>characterEncodingFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```

- 那么GET请求的中文乱码如何解决呢？
  - 在Tomcat配置文件的server.xml中的第一个`<Connector>`标签中加入：`URIEncoding="UTF-8"`即可



# 3.3 MyBatis

MyBatis中当实体类的属性名(如 lastName)和表中的字段名(如 last_name)不一样，怎么办？

- 方法一：写SQL语句时起别名

  ```sql
  select id,last_name lastName from employees where id = #{id}
  ```

- 方法二：在MyBatis的全局配置文件中开启驼峰命名规则：

  ```xml
      <settings>
          <setting name="mapUnderscoreToCamelCase" value="true"/>
      </settings>
  ```

- 方法三：在Mapper映射文件中使用resultMap来自定义映射规则

  ```xml
      <resultMap id="BaseResultMap" type="com.ms5cm.crud.bean.Employee">
          <id column="emp_id" jdbcType="INTEGER" property="empId"/>
          <result column="emp_name" jdbcType="VARCHAR" property="empName"/>
          <result column="gender" jdbcType="CHAR" property="gender"/>
          <result column="email" jdbcType="VARCHAR" property="email"/>
          <result column="d_id" jdbcType="INTEGER" property="dId"/>
      </resultMap>
  ```

