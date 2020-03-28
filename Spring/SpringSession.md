[Spring Session](https://spring.io/projects/spring-session)提供了一套创建和管理Servlet、HttpSession的方案，提供了集群Session（Clustered Sessions）功能，默认采用外置的Redis来存储Session数据，以此来解决同域名下的多服务器集群Session共享问题，不能解决跨域session共享问题。

使用时可以参照：https://www.imooc.com/article/21377

Spring Session项目集成。

# 实现同域下的SSO

1. 引入Spring Session；

2. 在web.xml中配置`DelegatingFilterProxy`；实现数据的拦截保存，并转换为Spring Session需要的会话对象

   ```xml
      <filter>
           <filter-name>springSessionRepositoryFilter</filter-name>
           <filter-class>org.springframework.web.filter.DelegatingFilterProxy</filter-class>
       </filter>
       <filter-mapping>
           <filter-name>springSessionRepositoryFilter</filter-name>
           <url-pattern>*.do</url-pattern>
       </filter-mapping>
   ```

3. 在Spring配置文件中配置`RedisHttpSessionConfiguration`、`DefaultCookieSerializer`来修改Cookie的默认设置

   ```xml
   <bean id="redisHttpSessionConfiguration" class="org.springframework.session.data.redis.config.annotation.web.http.RedisHttpSessionConfiguration">
       <!--默认的Session有效期也是1800s即30min-->
       <property name="maxInactiveIntervalInSeconds" value="1800"/>
   </bean>
   
       <bean id="defaultCookieSerializer" class="org.springframework.session.web.http.DefaultCookieSerializer">
           <property name="cookieName" value="SESSION_NAME"/>
           <property name="domainName" value=".fms5cms.com"/>
           <property name="useHttpOnlyCookie" value="true"/>
           <property name="cookiePath" value="/"/>
           <property name="cookieMaxAge" value="31536000"/>
       </bean>
   ```

4. 在Spring配置文件中配置`JedisConnectionFactory`，该连接工厂应使用环境配置和客户端配置来进行配置：

   1. 环境配置可以使用`RedisStandaloneConfiguration`，用于配置Redis的Host、port、password
   2. 客户端配置使用`JedisClientConfiguration`接口的子类来配置：
      - `JedisConnectionFactory`的内部类`MutableJedisClientConfiguration`就是上面接口的子类，
      - Redis的其他配置由三层架构构成,分别为:`BaseObjectPoolConfig` -> `GenericObjectPoolConfig` -> `JedisPoolConfig`
      - 实际上在`JedisPoolConfig`已经将所有配置设置完毕,如果用户需要自定义设置,可以创建`JedisPoolConfig`实例,也可以创建`GenericObjectPoolConfig`实例;
      - 如果用户未自定义`JedisPoolConfig`则`JedisConnectionFactory`直接使用`MutableJedisClientConfiguration`中的配置
      - 如果用户自定义了`JedisPoolConfig`,则会调用`setPoolConfig(GenericObjectPoolConfig poolConfig)`方法进行处理

   ```xml
   <bean id="redisStandaloneConfiguration" class="org.springframework.data.redis.connection.RedisStandaloneConfiguration">
       <property name="hostName" value="47.92.*.*"/>
       <property name="port" value="6379"/>
   </bean>
   
   <bean id="jedisConnectionFactory" class="org.springframework.data.redis.connection.jedis.JedisConnectionFactory">
       <constructor-arg name="standaloneConfig" ref="redisStandaloneConfiguration"/>
   </bean>
   ```

   



