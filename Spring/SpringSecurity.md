# 基于角色的权限管理

角色是代表一系列行为或责任的实体。用于限定用户在系统中的操作。用户账号往往和角色相关联。

- RBAC(Role-Based Access Control)：基于角色的访问控制。通常有隐式和显式两种访问控制方式。
  - 隐式访问控制：和特定角色相关联

    ```java
    if(user.hasRole("Project Manager")){
        //显示报表按钮
    } else {
        //不显示按钮
    }
    ```

  - 显式访问控制：判断是否有某种权限

    ```java
    if(user.isPermitted("projectReport:view:12345")){
        //显示报表按钮
    } else {
        //不显示按钮
    }
    ```

在Java平台中基于角色的权限管理的解决方案：Apache Shiro、Spring Security



# Spring Security

Spring Security是一个能够为基于Spring的企业应用系统提供**声明式的安全访问控制**解决方案的安全框架。它提供了一组可以在Spring应用上下文中配置的Bean，充分利用了Spring IoC，DI和AOP（面向切面编程）功能，为应用系统提供声明式的安全访问控制功能，减少了为企业系统安全控制编写大量重复代码的工作。

Spring Security 也是Spring Boot底层安全模块默认的技术选型。它可以实现强大的web安全控制。对于安全控制，我们仅需引入spring-boot-starter-security模块，进行少量的配置，即可实现强大的安全管理。

核心概念：

- 认证(Authentication)：建立主体(principal)的过程。
  - 主体通常指可以在应用程序中执行操作的用户、设备或其他系统
- 授权(Authorization)：也称为访问控制(access-control)，验证某个已认证的用户是否拥有某个权限；即判断用户是否能进行什么操作

Spring Security的身份验证技术：HTTP BASIC、HTTP Digest、HTTP X.509、LDAP、基于表单的认证、OpenID、单点登录、Remember-Me、Run-as、JAAS、JavaEE 容器认证。

Spring Security的模块：Core、Remoting、Web、Config、LDAP、ACL、CAS、OpenID、Test



# SpringBoot整合

[SpringBoot整合Security使用说明](https://docs.spring.io/spring-security/site/docs/current/guides/html5/helloworld-boot.html)

1. 引入依赖：

   ```xml
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-security</artifactId>
   </dependency>
   ```

2. 编写安全配置类（要继承`WebSecurityConfigurerAdapter`，并标注`@EnableWebSecurity`注解！！）。

   1. 在配置类中配置请求的访问权限：

      ```java
          @Override
          protected void configure(HttpSecurity http) throws Exception {
              //定制请求的授权规则
              http.authorizeRequests()
                      //访问"/"的请求允许所有人请求
                      .antMatchers("/").permitAll()
                      // "/level1"下的所有请求必须有VIP1的角色才能访问
                      .antMatchers("/level1/**").hasRole("VIP1")
                      .antMatchers("/level2/**").hasRole("VIP2")
                      .antMatchers("/level3/**").hasRole("VIP3");
      
              //开启自动配置的登录功能:
              //1. /login来到默认的登录页面，可以通过loginPage("")指定自定义的登录页面的请求路径
              //2. 重定向到 /login?error 表示登录失败
              //3. 默认POST形式的 /login 代表处理登录，默认使用username作为表单参数名来携带参数，如果要修改可以通过.usernameParameter("")来修改，同理可修改密码的默认参数名；GET形式的 /login 代表去登录页面
              //4. 一但定制 loginPage ，那么 loginPage 的POST请求就是登录
              //还有很多其他功能
              http.formLogin().loginPage("/userlogin");
              
              //开启自动配置的注销功能
              //1. 访问/logout表示用户注销、清空session
              //2. 注销成功会返回 /login?logout
              //3. 通过 logoutSuccessUrl("") 来配置注销成功后返回到哪个页面
              http.logout().logoutSuccessUrl("/");
              
              //开启记住我功能
              //登录成功后，将cookie发送给浏览器保存，以后访问页面带上这个cookie，只要通过检查就可以免登录，点击注销会删除cookie
              http.rememberMe();
          }
      ```

   2. 在配置类中定义认证规则:

      ```java
          @Override
          protected void configure(AuthenticationManagerBuilder auth) throws Exception {
              auth.inMemoryAuthentication() //认证信息存储在内存中
                  	//这里初始化了两个用户
                      .withUser("zzk").password("123456").roles("VIP1","VIP2")
                      .and()
                      .withUser("bx").password("123456").roles("VIP2");
          }
      ```

      

# Thymeleaf支持

Spring Security 整合 Thymeleaf 的地址：https://github.com/thymeleaf/thymeleaf-extras-springsecurity

```xml
<dependency>
    <groupId>org.thymeleaf.extras</groupId>
    <artifactId>thymeleaf-extras-springsecurity4</artifactId>
</dependency>
```

前端页面中引入 sec 标签：`xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity5"`。

- `sec:authentication="name"` 获得当前用户的用户名
- `sec:authorize="hasRole('ADMIN')"`当前用户必须拥有ADMIN权限时才会显示标签内容

```html
<!DOCTYPE html>
<html xmlns:th="http://www.thymeleaf.org"
	  xmlns:sec="http://www.thymeleaf.org/thymeleaf-extras-springsecurity4">
<head>
<meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
<title>Insert title here</title>
</head>
<body>
<h1 align="center">欢迎光临武林秘籍管理系统</h1>
<div sec:authorize="!isAuthenticated()">
	<h2 align="center">游客您好，如果想查看武林秘籍 <a th:href="@{/userlogin}">请登录</a></h2>
</div>
<div sec:authorize="isAuthenticated()">
	<h2><span sec:authentication="name"></span>，您好,您的角色有：
		<span sec:authentication="principal.authorities"></span></h2>
	<form th:action="@{/logout}" method="post">
		<input type="submit" value="注销"/>
	</form>
</div>
<hr>

<div sec:authorize="hasRole('VIP1')">
	<h3>普通武功秘籍</h3>
	<ul>
		<li><a th:href="@{/level1/1}">罗汉拳</a></li>
		<li><a th:href="@{/level1/2}">武当长拳</a></li>
		<li><a th:href="@{/level1/3}">全真剑法</a></li>
	</ul>
</div>

<div sec:authorize="hasRole('VIP2')">
	<h3>高级武功秘籍</h3>
	<ul>
		<li><a th:href="@{/level2/1}">太极拳</a></li>
		<li><a th:href="@{/level2/2}">七伤拳</a></li>
	</ul>
</div>

<div sec:authorize="hasRole('VIP3')">
	<h3>绝世武功秘籍</h3>
	<ul>
		<li><a th:href="@{/level3/2}">龟派气功</a></li>
		<li><a th:href="@{/level3/3}">独孤九剑</a></li>
	</ul>
</div>
</body>
</html>
```

