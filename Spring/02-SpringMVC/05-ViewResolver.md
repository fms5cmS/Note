# 5.0 视图解析器

请求处理方法执行完成后，最终会返回一个`ModelAndView`对象。对于哪些返回`String`、`View`、`ModelMap`等类型的处理方法，SpringMVC也会在内部把它们装配成一个`ModelAndView`对象，它包含逻辑名和模型对象的视图。

- 视图：渲染模型数据，将模型中的数据以某种形式呈给客户。

  - 为实现视图模型和具体实现技术的解耦，Spring在`org.springframework.web.servlet`包中定义了`View`接口；
  - **视图对象由视图解析器负责实例化**。由于视图是**无状态**的，所以**不会有线程安全的问题**。
  - 常用的是`InternalResourceView`、文档视图`AbstractExcelView`

- 视图解析器(ViewResolver)：将逻辑视图转为物理视图

  - 可以配置一种或多种解析策略，并通过order属性指定他们之间的先后顺序。每一种映射策略对应一个具体的视图解析器实现类。
  - 所有视图解析器都必须实现`ViewResolver`接口，返回一个`View`实例。
  - `InternalResourceViewResolver`是最常用的 JSP 的视图解析器。

- 若希望直接响应通过 SpringMVC 渲染的页面，可以在SpringMVC配置文件中使用\<mvc:view-controller>标签实现。

  ```xml
  <!-- 直接配置响应的页面：无需经过控制器来执行结果 -->
  <mvc:view-controller path="/success" view-name="success"/>
  <!--配置<mvc:view-controller>会导致其他请求路径失效。解决办法：配置以下标签-->
  <mvc:annotation-driven/>
  ```



# 5.1 JSP视图相关

Spring提供了两种支持JSP视图的方式：

1. `InternalResourceViewResolver`会将视图名解析为`InternalResourceView`实例。
   - 若JSP页面中使用了JSTL，该解析器会把视图名转为`JstlView`实例，从而将JSTL本地化和Resource Bundle变量暴露给JSTL的格式化和信息标签。
2. Spring提供了两个JSP标签库，一个用于表单到模型的绑定，一个提供了通用的工具类特性。



## 配置JSP视图解析器

`InternalResourceViewResolver`在解析视图名时，会在视图名上添加前缀和后缀，进而确定一个Web应用中视图资源的物理路径。

下面在配置类中指定了`InternalResourceViewResolver`，其中前缀设置为`/WEB-INF/views/`，后缀设置为`.jsp`。假设控制器对某个请求处理后，返回的视图名为`home`，那么，经过`InternalResourceViewResolver`解析后，实际的物理视图即为：`/WEB_INF/views/home.jsp`。

```java
@Bean
public ViewResolver viewResolver(){
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    //这里通常用于设置视图所在的目录
    resolver.setPrefix("/WEB-INF/views/");
    //这里用于设置视图的后缀。视图名由控制器的方法返回
    resolver.setSuffix(".jsp");
    return resolver;
}
```

当然，也可以使用配置文件的方式来配置视图解析器，SpringMVC配置文件中：

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/views/hello/"/>
    <property name="suffix" value=".jsp"/>
</bean>
```

注意：当控制器返回的逻辑视图名中 包含`/`时，`/`也会被带到资源的路径名中。假设逻辑视图名为`books/detail`，则解析后的物理视图为`/WEB_INF/views/books/detail.jsp`。



## 解析JSTL视图

- 若JSP页面使用了JSTL来处理格式化和信息的话，那么希望`InternalResourceViewResolver`将视图名解析为`JstlView`实例。
  - JSTL的格式化标签需要一个`Local`对象，以便恰当地格式化地域相关的值，如：日期、货币。
  - JSTL的信息标签可以借助Spring的信息资源和`Local`，从而选择适合的信息渲染到HTML中。
- 通过解析`JstlView`，JSTL能够获得`Local`对象及Spring中配置的信息资源。

为了让`InternalResourceViewResolver`将视图名解析为`JstlView`实例，需要设置它的`viewClass`属性！！

```java
@Bean
public ViewResolver viewResolver(){
    InternalResourceViewResolver resolver = new InternalResourceViewResolver();
    resolver.setPrefix("/WEB-INF/views/");
    resolver.setSuffix(".jsp");
    resolver.setViewClass(JstlView.class);
    return resolver;
}
```

配置文件的方式：

```xml
<bean class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/views/hello/"/>
    <property name="suffix" value=".jsp"/>
    <property name="viewClass" value="org.springframework.web.servlet.view.JstlView"/>
</bean>
```



## Spring的JSP标签库

- Spring提供了两个JSP标签库，用于帮助定义SpringMVC Web的视图
  - `<form>`标签库用于渲染HTML**表单标签**，**可以绑定模型中的某个属性**；
  - `<spring>`标签库包含了一些工具类标签



### form标签库

可以实现将模型数据中的属性和HTML表单元素相绑定，以实现表单数据更**便捷编辑和表单值的回显**。

一般情况下，**通过GET请求获取表单页面，而通过POST请求提交表单页面，因此获取表单页面和提交表单页面的URL是相同的**。只要满足该最佳条件，`<form:form>`标签就无需通过`action`属性指定表单提交的URL。

引入form标签库：

```jsp
<%@ taglib prefix="form" uri="http://www.springframework.org/tags/form" %>
```

所有标签：

| JSP标签               | 描述                                                         |
| --------------------- | ------------------------------------------------------------ |
| `<form:checkbox>`     | 复选框组件。渲染成一个HTML`<input>`标签，其中`type=checkbox` |
| `<form:checkboxs>`    | 构造多个复选框。渲染成多个HTML`<input>`标签，其中`type=checkbox` |
| `<form:errors>`       | 显示表单组建或数据校验所对应得错误。在一个HTML`<span>`中渲染输入域的错误 |
| `<form:form>`         | 渲染成一个HTML`<form>`标签，并为内部标签暴露绑定路径，用于数据绑定 |
| `<form:hidden>`       | 渲染成一个HTML`<input>`标签，其中`type=hidden`               |
| `<foem:input>`        | 渲染成一个HTML`<input>`标签，其中`type=text`                 |
| `<form:label>`        | 渲染成一个HTML`<label>`标签                                  |
| `<form:option>`       | 下拉框，渲染成一个HTML`<option>`标签，其中`selected`属性根据所绑定的值进行设置 |
| `<form:options>`      | 按照绑定的集合、数组或Map，渲染成一个HTML`<option>`标签的列表 |
| `<form:password>`     | 渲染成一个HTML`<input>`标签，其中`type=password`             |
| `<form:radiobutton>`  | 单选框组件，渲染成一个HTML`<input>`标签，其中`type=radio`    |
| `<form:radiobuttons>` | 单选框组，渲染成多个HTML`<input>`标签，其中`type=radio`      |
| `<form:select>`       | 渲染成一个HTML`<select>`标签                                 |
| `<form:textarea>`     | 渲染成一个HTML`<textarea>`标签                               |



```jsp
    <form:form method="post" commandName="spitter"><!--commandName指定模型对象为spitter-->
        Name:<form:input path="name"/> <br/>
        Password:<form:password path="password"/> <br/>
        <input type="submit" value="Register">
    </form:form>
```

- `<form:form>`会渲染成一个HTML`<form>`标签，会通过`commandName`属性构建针对某个模型对象的上下文信息。其他的表单标签中，会通过各自的`path`属性引用这个模型对象的对应属性。
- SpringMVC提供了多个表单组件标签，如`<form:input>`、`<form:select>`等，用以绑定表单字段的属性值，它们的共有属性如下：
  - **`path`：表单字段，对应html表单标签的name属性，支持级联属性**
  - `htmlEscape`：是否对表单值得HTML特殊字符进行转换，默认为true
  - `cssClass`：表单组件对应得CSS样式类名
  - `caaErrorClass`：表单组建的数据存在错误时，采用的CSS样式



表单值回显：在校验失败后，表单中会预先填充之前输入的值，但并没有说明哪里出错，此时可以使用`<form:errors>`。

- `<form:errors>`：显示表单组建或数据校验所对应得错误
  - `<form:errors path="*">`：显示表单所有错误
  - `<form:errors path="user*">`：显示以user为前缀的属性对应得错误
  - `<form:errors path="username">`：显示特定表单对象属性的错误

```html
<form:form method="post" commandName="spitter">
    Name:<form:input path="name"/> <br/>
    <form:errors path="name"/> <br/>
    Password:<form:password path="password"/> <br/>
    <input type="submit" value="Register">
</form:form>
```

上面的例子在第三行使用了`<form:errors>`标签，且`path="name"`，即指定了要显示Spitter模型对象中name属性的错误，如果该属性没有错误，则不会渲染任何内容，如果有错误，则会用一个HTML`<span>`标签中显示错误信息。

上面的错误信息会显示在各自错误出现的地方，如果想要将所有错误信息在同一个地方显示，可以移除每个输入域的`<form:errors>`标签，而仅在表单的顶部放入该标签：`<form:errors path="*" element="div"/>`，注意：`path=*`表示会显示属性所有错误；当有多个错误时，默认显示在HTMl`<span>`标签中，此时不太合适，故使用`element`属性指定错误渲染在一个`<div>`标签中。



### spring标签库

引入标签库：

```xml
<%@ taglib prefix="spring" uri="http://www.springframework.org/tags" %>
```

| JSP标签                  | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `<spring:escapeBody>`    | 将标签体中的内容进行HTML和JavaScrit转义                      |
| `<spring:hasBindErrors>` | 根据指定模型对象是否有绑定错误，有条件地渲染内容             |
| `<spring:message>`       | 给句请求的编码获取信息，然后要么进行渲染(默认)，要么将其设置为页面作用域、请求作用域、会话作用域或应用作用域地变量(使用var和scope属性实现) |
| `<spring:nestedPath>`    | 设置嵌入式地path，用于`<spring:bind>`中                      |
| `<spring:theme>`         | 根据给定地编码获取主题信息，然后要么进行渲染(默认)，要么将其设置为页面作用域、请求作用域、会话作用域或应用作用域地变量(使用var和scope属性实现) |
| `<spring:transform>`     | 使用命令对象地属性编辑器转换命令对象中不包含地属性           |
| `<spring:url>`           | 创建相对于上下文地URL，支持URI模板变量及HTML/XML/JavaScript转义，可渲染URL(默认行为)，也可将其设置为页面作用域、请求作用域、会话作用域或应用作用域地变量(使用var和scope属性实现) |
| `<spring:eval>`          | 计算符合SpEL语法地某个表达式的值，然后要么进行渲染(默认)，要么将其设置为页面作用域、请求作用域、会话作用域或应用作用域地变量(使用var和scope属性实现) |
| `<spring:htmlEscape>`    | 为当前页面设置默认地HTML转义值                               |



展示国际化信息：

- `<spring:message>`用于渲染文本很合适

  ```jsp
  <h1><spring:message code="spittr.welcome"></h1>
  ```

  - 该标签会根据 key=spittr.welcome 的信息源来渲染文本。因此需要配置一个信息源。信息源都实现了`MessageSource`接口，常用的是`ResourceBundleMessageSource`，它会从一个属性文件中加载信息，这个属性文件的名称是根据基名衍生来的：

    ```java
        @Bean
        public MessageSource messageSource(){
            ResourceBundleMessageSource messageSource = new ResourceBundleMessageSource();
            messageSource.setBasename("i18n");
            return messageSource;
        }
    ```

  - 还有一种可选方案是使用`ReloadableResourceBundleMessageSource`，其工作方式与`ResourceBundleMessageSource`类似，但可以重新加载信息属性，而不用重新编译或重启应用：

    ```java
        @Bean
        public MessageSource messageSource(){
            ReloadableResourceBundleMessageSource messageSource = new ReloadableResourceBundleMessageSource();
            messageSource.setBasename("file:///etc/spittr/i18n");
            messageSource.setCacheSeconds(10);
            return messageSource;
        }
    ```

    - 类路径查找：`classpath:`；文件系统查找：`file:`。

  - 属性文件的配置：i18n.properties

    ```properties
    spittr.welcome
    ```



创建URL：

- `<spring:url>`创建URL，然后将其赋值给一个变量或渲染到响应中

  - `<spring:url>`会接受一个相对于上下文的URL，并在渲染的时候，预先添加上Servlet上下文路径：

    ```jsp
     <a href="<spring:url href="/spitter/register"/>">Register</a>
    ```

    - 如果应用的上下文名是spittr，那么在响应中会渲染如下的HTML：

      ```html
      <a href="/spittr/spitter/register">Register</a>
      ```

  - 使用`<spring:url>`创建URL,并将其赋值给一个变量供模版在稍后使用：

    ```jsp
    <spring:url value="/spitter/register" var="registerUrl" />
    <a href="${registerUrl}"></a>
    ```

    - 默认情况下，URL是在页面作用于内创建的。但是可以通过scope属性，让`<s:url>`在应用作用域内、会话作用域内或者请求作用域内创建URL：

      ```jsp
      <spring:url value="/spitter/register" var="registerUrl"  scope="request"/>
      ```

  - 在URL上添加参数，可以使用`<spring:param>`标签：

    ```jsp
     <spring:url value="/spittles" var="spittlesUrl">
            <spring:param name="max" value="60" />
            <spring:param name="count" value="20" />
     </spring:url>
    ```

    - 使用`<s:param>`标签还可以创建带有路径参数的URL，使其具有路径变量的占位符：

      ```jsp
      <spring:url value="/spittles/{userName}" var="spittlesUrl">
              <spring:param name="userName" value="jbauer" />
       </spring:url>
      ```

    - 当value属性中的占位符匹配`<spring:param>`中指定的参数时，这个参数将会插入到占位符的位置中。如果`<spring:param>`参数无法匹配value中的任何占位符，那么这个参数将会作为查询参数

  - `<spring:url>`还可以解决URL的转移需求。

    - 可以通过设置htmlEscape属性为true，将渲染得到的URL内容展现在Web页面上（而不是作为超链接）：

      ```jsp
      <spring:url value="/spittles" htmlEscape="true">
              <spring:param name="max" value="60" />
              <spring:param name="count" value="20" />
      </spring:url>
      ```

    - 可以将javaScriptEscape属性为true，便可以在Java代码中使用URL：

      ```jsp
      <spring:url value="/spittles" var="spittlesJSUrl" javaScriptEscape="true">
            <spring:param name="max" value="60" />
            <spring:param name="count" value="20" />
       </spring:url>
       <script>
            var spittlesUrl = "${spittlesJSUrl}"
       </script>
      ```



转义内容：

- `<spring:escapeBody>`标签是一个通用的转义标签。通过设置htmlEscape属性为true，可以在页面上展现一个HTML代码片段；通过设置javaScript属性为true，可以实现JavaScript的的转义。

 