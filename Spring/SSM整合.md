# SSM整合

1. 导入jar包；

2. 编写配置文件：

   - web.xml：配置Spring、SpringMVC；

     ```xml
     <!--1.启动Spring容器-->
     <context-param>
         <param-name>contextConfigLocation</param-name>
         <param-value>classpath:applicationContext.xml</param-value>
     </context-param>
     <!--如果不配置contextConfigLocation参数，这个监听器默认寻找 /WEB-INF/applicationContext.xml的配置文件来初始化bean-->
     <listener>
         <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
     </listener>
     
     <!--2.SpringMVC的前端控制器，拦截所有请求-->
     <servlet>
         <servlet-name>dispatcherServlet</servlet-name>
         <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
         <!--注意，这里没有指定springMVC配置文件的位置，所以采用了默认的方式-->
         <load-on-startup>1</load-on-startup>
     </servlet>
     <servlet-mapping>
         <servlet-name>dispatcherServlet</servlet-name>
         <url-pattern>/</url-pattern>
     </servlet-mapping>
     
     <!--3.字符编码过滤器，一定要放在所有过滤器之前-->
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
     
     <!--4.使用REST风格的URL,将页面的post请求转为delete或put请求-->
     <filter>
         <filter-name>hiddenHttpMethodFilter</filter-name>
         <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
     </filter>
     <filter-mapping>
         <filter-name>hiddenHttpMethodFilter</filter-name>
         <url-pattern>/*</url-pattern>
     </filter-mapping>
     
     <!--5.为支持直接发送PUT请求，配置该过滤器，可以将请求体中的数据解析包装为一个Map，request被重新包装，这样，request.getParameter()被重写，就可以从封装的Map中获取数据-->
     <filter>
         <filter-name>httpPutFormContentFilter</filter-name>
         <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
     </filter>
     <filter-mapping>
         <filter-name>httpPutFormContentFilter</filter-name>
         <url-pattern>/*</url-pattern>
     </filter-mapping>
     ```

   - SpringMVC的配置文件：

     ```xml
     <!--SpringMVC的配置文件,主要包含网站的跳转逻辑和配置-->
     <context:component-scan base-package="com.ms5cm" use-default-filters="false">
         <!--只让SpringMVC扫描控制器-->
         <context:include-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
     </context:component-scan>
     
     <!--配置视图解析器-->
     <bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
         <property name="prefix" value="/WEB-INF/views/"/>
         <property name="suffix" value=".jsp"/>
     </bean>
     
     <!--两个标准配置-->
     <!--将SpringMVC不能处理的请求交给Tomcat-->
     <mvc:default-servlet-handler/>
     <!--支持一些SpringMVC更高级的功能：JSR-303校验、快捷的Ajax、映射动态请求...-->
     <mvc:annotation-driven/>
     ```

   - Spring的配置文件：

     ```xml
     <!--Spring的配置文件，主要配置和业务逻辑相关的:数据源、事务控制等-->
     <!--Spring配置文件的核心点：数据源、与Mybatis整合、事务控制-->
     <context:component-scan base-package="com.ms5cm.crud">
         <context:exclude-filter type="annotation" expression="org.springframework.stereotype.Controller"/>
     </context:component-scan>
     
     <!--===============================数据源配置===================================-->
     <context:property-placeholder location="classpath:dbconfig.properties"/>
     
     <bean id="pooledDataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
         <property name="jdbcUrl" value="${jdbc.jdbcUrl}"/>
         <property name="driverClass" value="${jdbc.driverClass}"/>
         <property name="user" value="${jdbc.user}"/>
         <property name="password" value="${jdbc.password}"/>
     </bean>
     
     <!--=======================配置和Mybatis的整合=================================-->
     <bean id="sqlSessionFactory" class="org.mybatis.spring.SqlSessionFactoryBean">
         <!--指定Mybatis全局配置文件的位置-->
         <property name="configLocation" value="classpath:mybatis-config.xml"/>
         <property name="dataSource" ref="pooledDataSource"/>
         <!--指定Mybatis的mapper文件的位置-->
         <property name="mapperLocations" value="classpath:mapper/*.xml"/>
     </bean>
     
     <!--配置扫描器，将Mybatis接口的实现加入到IoC容器中-->
     <bean class="org.mybatis.spring.mapper.MapperScannerConfigurer">
         <!--扫描所有dao接口的实现，并加入到IoC容器中-->
         <property name="basePackage" value="com.ms5cm.crud.dao"/>
     </bean>
     
     <!--由于会经常使用，配置一个可以执行批量操作的sqlSession-->
     <bean id="sqlSession" class="org.mybatis.spring.SqlSessionTemplate">
         <constructor-arg name="sqlSessionFactory" ref="sqlSessionFactory"/>
         <constructor-arg name="executorType" value="BATCH"/>
     </bean>
     <!--=========================事务控制================================-->
     <!--配置事务控制的管理器-->
     <bean id="transactionManager" class="org.springframework.jdbc.datasource.DataSourceTransactionManager">
         <!--控制住数据源-->
         <property name="dataSource" ref="pooledDataSource"/>
     </bean>
     <!--开启基于注解的事务，也可以使用xml配置形式的事务(主要都是使用配置式)-->
     <aop:config>
         <!--切入点表达式-->
         <aop:pointcut id="txPoint" expression="execution(* com.ms5cm.crud.service..*(..))"/>
         <!--配置事务增强-->
         <aop:advisor advice-ref="txAdvice" pointcut-ref="txPoint"/>
     </aop:config>
     
     <!--配置事务增强，事务如何切入，注意：这里其实还有一个transaction-manager属性用于设置事务管理器，默认值为transactionManager-->
     <tx:advice id="txAdvice">
         <tx:attributes>
             <!--该切入点切入的所有方法都是事务方法-->
             <tx:method name="*"/>
             <!--以get开始的方法-->
             <tx:method name="get*" read-only="true"/>
         </tx:attributes>
     </tx:advice>
     ```

   - MyBatis配置文件：

     ```xml
     <?xml version="1.0" encoding="UTF-8" ?>
     <!DOCTYPE configuration
             PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
             "http://mybatis.org/dtd/mybatis-3-config.dtd">
     <configuration>
     
         <!-- 使用dbconfig.properties数据库配置文件 -->
         <properties resource="dbconfig.properties"></properties>
     
         <settings>
             <!--开启驼峰命名规则-->
             <setting name="mapUnderscoreToCamelCase" value="true"/>
         </settings>
     
         <typeAliases>
             <package name="com.ms5cm.crud.bean"/>
         </typeAliases>
     
         <plugins>
             <plugin interceptor="com.github.pagehelper.PageInterceptor">
                 <!--分页参数合理化：无法到达不正确的页码，如-1页-->
                 <property name="reasonable" value="true"/>
             </plugin>
         </plugins>
     </configuration>
     
     ```

