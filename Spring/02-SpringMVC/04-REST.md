# 4.0 REST认识

REST(Representational State Transfer)资源表现层状态转化。

资源（Resources）：每种资源对应一个特定的URI，URI即为每个资源独一无二的标识符；

表现层（Representation）：把资源具体呈现出来的形式，如：文本用txt格式表现，也可用HTML格式；通过Http协议中Headers的Content-Type和accept来指定展示形式

状态转化（State Transfer）：如果客户端想操作服务器，必须通过某种手段，让服务器端发生“状态转化”，这种转化是建立在表现层上的。具体讲，就是**HTTP协议中，4个表现操作方式的动作：GET、POST、PUT、DELETE，它们分别对应四种基本操作：GET用来获取资源、POST用来新建资源(不具有幂等性)、PUT用来新建(更新)资源、DELETE用来删除资源。**（幂等性：每次HTTP请求相同的参数、相同的URI，产生的结果是相同的）

URI： /资源名称/资源标识               HTTP请求方式区分对资源CRUD操作

| 功能                                 | 请求URI  | 请求方式 |
| ------------------------------------ | -------- | -------- |
| 查询所有员工                         | emps     | GET      |
| 查询某个员工(来到修改页面)           | emp/{id} | GET      |
| 来到添加页面                         | emp      | GET      |
| 添加员工                             | emp      | POST     |
| 来到修改页面（查出员工进行信息回显） | emp/{id} | GET      |
| 修改员工                             | emp      | PUT      |
| 删除员工                             | emp/{id} | DELETE   |

更多了解，见：[理解本真的REST架构风格](https://kb.cnblogs.com/page/186516/)、[深入浅出 REST](https://www.infoq.cn/article/rest-introduction)



# 4.1 两个过滤器

`HiddenHttpMethodFilter`：form表单只支持GET、POST请求，而DELETE、PUT等method并不支持，该过滤器可以把这些请求转换为标准的http方法，使得支持GET、POST、PUT、DELETE请求。

Tomcat 8以上版本运行时可进入到相应的控制器，但是视图渲染返回的时候，由于不支持 delete 和 post 这两种方法，就会报出异常页面。

在web.xml中配置：

```xml
<!--使用REST风格的URL,将页面的post请求转为delete或put请求-->
<filter>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <filter-class>org.springframework.web.filter.HiddenHttpMethodFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>hiddenHttpMethodFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>

<!--为支持直接发送PUT请求，配置该过滤器，可以将请求体中的数据解析包装为一个Map，request被重新包装，这样，
        request.getParameter()被重写，就可以从封装的Map中获取数据-->
<filter>
    <filter-name>httpPutFormContentFilter</filter-name>
    <filter-class>org.springframework.web.filter.HttpPutFormContentFilter</filter-class>
</filter>
<filter-mapping>
    <filter-name>httpPutFormContentFilter</filter-name>
    <url-pattern>/*</url-pattern>
</filter-mapping>
```



# 4.2 静态资源

优雅的 REST 风格的资源URL 不希望带 .html 或 .do 等后缀，若将`DispatcherServlet`请求映射配置为`/`, 则 Spring MVC 将捕获 WEB 容器的所有请求，包括静态资源的请求，SpringMVC 会将他们当成一个普通请求处理, 因找不到对应处理器将导致错误。为了使SpringMVC处理静态资源，可以在SpringMVC配置文件中增加`<mvc:default-servlet-handler/>`标签，但是这样的话，原来的请求又不好使了，所以再增加`<mvc:annotation-driven/>`标签：

```xml
<!--将SpringMVC不能处理的请求交给Tomcat-->
<mvc:default-servlet-handler/>
<!--支持一些SpringMVC更高级的功能：JSR-303校验、快捷的Ajax、映射动态请求...-->
<mvc:annotation-driven/>
```

`<mvc:default-servlet-handler/>`将在 SpringMVC 上下文中定义一个`DefaultServletHttpRequestHandler`，它会对进入 `DispatcherServlet`的请求进行筛查，如果发现是没有经过映射的请求，就将该请求交由 WEB 应用服务器默认的 Servlet 处理，如果不是静态资源的请求，才由`DispatcherServlet`继续处理。

一般 WEB 应用服务器默认的 Servlet 的名称都是 default。若所使用的 WEB 服务器的默认 Servlet 名称不是 default，则需要通过`<mvc:annotation-driven />`的default-servlet-name 属性显式指定。



# 4.3 处理JSON

使用消息转换器将数据转换为JSON(消息转换见3.3)。如果要处理JSON需要加入jackson-databind包(由于Maven会自动将jackson-databind依赖的jackson-annotation、jackson-core，所以没有显式地配置这两个jar包)：

```xml
<dependency>
    <groupId>com.fasterxml.jackson.core</groupId>
    <artifactId>jackson-databind</artifactId>
    <version>2.8.8</version>
</dependency>
```



## 响应体

输出JSON数据给客户端。

正常情况下，当处理方法返回Java对象时，这个对象会放在模型中并在视图中渲染使用。但如果使用了消息转换功能，为了告诉Spring**跳过正常的模型/视图流程，并使用消息转换器**，可以为控制器方法**添加`@ResponseBody`注解**。

```java
@RequestMapping(value = "/emp/{id}",method = RequestMethod.GET，produces="application/json")
@ResponseBody
public Msg getEmp(@PathVariable("id") Integer id){
    Employee employee = employeeService.getEmp(id);
    return Msg.success().add("emp",employee);
}
```

`@ResponseBody`注解告诉Spring，要将返回的对象作为资源发送给客户端，并将其转换为客户端可接受的表述形式。具体来说，`DispacterServlet`会考虑到请求头中Accept头部信息，并查找能为客户端提供所需表述形式的消息转换器。

如：假设客户端的Accept头部信息表明它接受“application/json”，且jackson库位于应用的类路径下，那么将会选择`MappingJacksonHttpMessageConverter`或`MappingJackson2HttpMessageConverter`(取决于jackson库的版本)，该消息转换器将控制器返回的数据转换为JSON文档，并写入到响应体中。

注意：默认情况下，jackson库在将返回的对象转换为JSON资源表述时，会使用反射。

可以使用`@RequsetMapping`注解的produces属性来声明该方法只处理预期输出为JSON的请求，这样即使URL匹配指定的路径且是GET请求也不会被这个方法处理。

将返回类型设置为`ResponseEntity`类也可以开启消息转换，不过一般不会使用，因为通常会使用一个自定义的高复用的服务响应对象作为返回类型。



## 请求体

`@RequestBody`注解可以将来自客户端的资源表述转为对象。该注解**标注在处理器方法对应参数**上。

Spring会查看请求中的 Content-Type 头部信息，并查找能将请求体转为对应参数的消息转换器。

`@RequsetMapping`注解的consumes属性可以指定要处理的请求的Content-Type头部信息的具体内容。

```java
@ResponseBody
@RequestMapping(value = "/emp/{empId}",method = RequestMethod.PUT,consumes="application/json")
public Msg updateEmp(@RequestBody Employee employee){
    System.out.println("将要更新的员工数据："+employee);
    employeeService.updeteEmp(employee);
    return Msg.success();
}
```

也可以将参数设置为`HttpEntity`类来开启消息转换，不过一般不会使用。



## 综合

上面的两个注解是启用消息转换的一种很强大简洁的方式，但如果控制器有很多方法，而它们都需要使用消息转换，那么这些注解会带来一定重复性。

在**控制器类上使用`@RestController`**来替代`@Controller`的话，Spring会为该控制器的**所有处理方法**应用消息转换功能。

