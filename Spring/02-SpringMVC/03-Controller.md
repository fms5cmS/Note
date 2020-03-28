# 3.1 @RequestMapping

- `@RequestMapping`注解可以标注在控制器的类、方法处
  - 类：提供初步的请求映射信息。相对于WEB应用的根目录；
  - 方法：提供进一步的细分映射信息。相对于上面类处的URL。
    - 若类处未标注该注解，则方法处标记的URL是相对于WEB应用的根目录。

`DispatcherServlet`拦截请求后，通过控制器上的`@RequestMapping`提供的映射信息确定请求所对应的处理方法。

- `@RequestMapping`注解除了可以使用**请求URL**映射请求外，还可用**请求方法、请求参数、请求头**来映射请求
  - 注解的**`value`、`method`**、`params`、`headers`分别表示上面四种映射方式，常用前两种。
  - 一起使用的话，它们之间是**与**的关系，联合使用多个条件使请求映射精确化
  - **`method`属性使用`RequestMethod`类的常量来赋值，如：`@RequestMapping(method=RequestMethod.GET)`**
  - `params`和`headers`（了解即可）支持简单的表达式，如：
    - `param1`：表示请求必须包含名为param1的请求参数
    - `!param1`：表示请求不能包含
    - `param1 != value1`：表示请求包含名为param1的请求参数，但其值不能为value1
    - `{"param1=value1","param2"}`：请求必须包含名为param1、param2的请求参数，且param1值为value1
- 该注解在映射请求时支持Ant风格URL的通配符：
  - `?`：匹配文件名中一个字符
  - `*`：匹配文件名中任意长的任意字符
  - `**`：匹配多层路径

```java
@Controller
@RequestMapping({"/","/homepage"})
public class HomeController {
    @RequestMapping( method = RequestMethod.GET)
    public String home() {
        return "home";	//返回视图名
    }
}
```

针对不同的请求方式，也可以使用`@PostMapping`、`@GetMapping`等注解！



# 3.2 接收请求参数

- SpringMVC允许以多种方式将客户端数据传送到控制器的处理方法中，包括：
  - 查询参数
  - 表单参数
  - 路径变量



## 查询参数

- `@RequestParam`注解用于绑定请求参数值
  - 标注在方法入参上，把请求参数传递给请求方法
  - 属性：
    - `vale`：请求参数的参数名
    - `required`：默认为true，表示请求参数中必须包含对应参数，若不存在，抛出异常
    - `defaultValue`：用于在URL中不存在参数时给处理方法传递一个默认值。如果参数为引用类型，默认值为`null`，如果参数为基本数据类型，必须使用该属性指定默认值。

```java
@RequestMapping("/testRequestParam")
public String testRequestParam(@RequestParam("user") String user,
                               @RequestParam(value = "age",defaultValue = "18") int age
                               @RequestParam(value = "id",required = false) Integer id){
    System.out.println("user:"+user+";age:"+age);
    return "success";
}
```

可以获取URL：`http://localhost:8080/testRequestParam?user-zzk&age=22&id=28`的参数。



## 路径变量

在理想情况下，要识别的资源应通过URL路径进行标示，而不是通过查询参数。如：“/hello/23”发送GET请求要优于“/hello/source_id=23”发起请求。

通过`@PathVariable`注解可以将URL中的占位符参数绑定到控制器处理方法的入参中。常用于REST中获取参数。

如果`@PathVariable`中没有value属性的话，默认占位符的名称与方法参数名相同。

```java
//此时可以处理如：/hello/23 这样直接将参数跟在路径后面而不使用键值对方式
@RequestMapping("/hello/{id}")
public String hello(@PathVariable("id") Integer id){
    System.out.println(id);
    System.out.println("hello SpringMVC");
    return "success";
}
```

Spring 4.3.3开始，`@PathVariable`新增了required属性来指定该路径变量是否为必须，默认为true。



## 表单参数

SpringMVC会**按照参数名和POJO属性名进行自动匹配，自动为该对象填充属性值。支持级联属性**，如：dept.deptId。

在页面中的表单：

```jsp
    <form action="/testPojo" method="post">
        id:<input type="text" name="id"/>
        username:<input type="text" name="username"/>
        age:<input type="text" name="age">
        <!--注意：这里的级联属性中address为User类中持有的Address类的对象名-->
        city:<input type="text" name="address.city">
        <input type="submit" value="Submit">
    </form>
```

```java
@RequestMapping("/testPojo")
public String testPojo(User user) {
    System.out.println("testPojo: " + user);
    return "success";
}
```



## Servlet原生API

- SpringMVC的Handler方法可以接收的Servlet API：
  - `HttpServletRequest`
  - `HttpServletResponse`
  - `HttpSession`
  - `java.security.Principal`
  - `Locale`
  - `InputStream`
  - `OutputStream`
  - `Reader`
  - `Writer`

```java
@RequestMapping("/testServletAPI")
public void testServletAPI(HttpServletRequest request,HttpServletResponse response, 
                           Writer out) throws IOException {
    System.out.println("testServletAPI, " + request + ", " + response);
    out.write("hello springmvc");
}
```



## Cookie&Header

- `@RequestHeader`注解可将请求头中的属性值绑定到处理方法的入参中。了解
  - 属性与`@RequestParam`是一样的，用法相同。请求头信息是浏览器设置的，不用在超链接中加
- `@CookieValue`注解用于绑定请求中的Cookie值。了解
  - 用法和`@RequestParam`类似



# 3.3 消息转换

消息转换(message conversion)可以将控制器产生的数据转换为服务于客户端的表达形式。当使用消息转换功能时，`DispacterServlet`不再将模型数据传送到视图中，实际上，这里根本没有模型，也没有视图，只有控制器产生的数据，以及消息转换器转换数据后产生的资源表述。

Spring自带了许多HTTP信息转换器，用于实现资源表述与各种Java类型之间的互相转换。这些HTTP信息转换器大部分都是自动注册的，所以使用时不需要Spring配置，但为了支持它们，需要添加一些库(jar包)到应用程序的类路径下。如：要使用`MappingJackson2HttpMessageConverter`实现JSON消息和java对象互相转换，需要将jackson-databind包放入类路径下，见4.3。

`HttpMessageConverter`接口，将请求信息转换为一个对象，并将对象绑定到请求方法的参数中或输出为响应信息。

`DispatcherServlet`默认已经装配了`RequestMappingHandlerAdapter`作为`HandlerAdapter`组件的实现类，即`HttpMessageConverter`由`RequestMappingHandlerAdapter`使用，将请求信息转为对象或将对象转为响应信息。所以我们通常不需要显式地指定处理适配器。

而如果在SpringMVC配置文件中显式地定义一个`RequestMappingHandlerAdapter`，则SpringMVC地`RequestMappingHandlerAdapter`默认装配的`HttpMessageConverter`不再起作用！



# 3.4 数据格式化(了解)

 Spring 在格式化模块中定义了一个实现`ConversionService`接口的**`FormattingConversionService`**实现类，该实现类扩展了`GenericConversionService`，因此它既有类型转换的功能，又有格式化的功能。

`FormattingConversionService`拥有一个`FormattingConversionServiceFactoryBean`工厂类 ， 后者用于在Spring 上下文中构造前者 ，`FormattingConversionServiceFactoryBean`内部已经注册了：

1. `NumberFormatAnnotationFormatterFactory`：支持对数字类型的属性使用`@NumberFormat`注解；
2. `JodaDateTimeFormatAnnotationFormatterFactory`：支持对日期类型的属性使用`@DateTimeFormat`注解。

装配了`FormattingConversionServiceFactroyBean`后，就可在Spring MVC 入参绑定及模型数据输出时使用注解驱动了。

<u>注：SpringMVC配置文件中使用\<mvc:annotation-driven/> 标签默认创建的`ConversionService`实例即为`DefaultFormattingConversionService`。</u>

- 日期格式化：`@DateTimeFormat`注解可对`java.util.Date`、`java.util.Calendar`、`java.long.Long`时间
  类型进行标注；
- 数值格式化：`@NumberFormat`可对类似数字类型的属性进行标注。





# 3.5 数据校验

使用Java检验API(Java Validation API，也称JSR-303)完成。

Spring 在进行数据绑定时，可同时调用校验框架完成数据校验工作。 在 Spring MVC中，**可直接通过注解驱动的方式进行数据校验**。

 Spring 的`LocalValidatorFactroyBean`既实现了 Spring 的`Validator`接口，也实现了JSR-303 的`Validator`接 口 。 只 要在Spring容器中定义了一个`LocalValidatorFactoryBean`，即可将其注入到需要数据校验的 Bean 中。

SpringMVC配置文件的\<mvc:annotation-driven/> 会默认装配好一个`LocalValidatorFactoryBean`，通过在处理方法的入参上标注`@Valid`注解即可让 SpringMVC 在完成数据绑定后执行数据校验的工作。



使用步骤：

1. 加入 hibernate validator 验证框架的 jar 包

   ```xml
   <dependency>
       <groupId>org.hibernate</groupId>
       <artifactId>hibernate-validator</artifactId>
       <version>6.0.12.Final</version>
   </dependency>
   ```

2. 在 bean 的属性上添加对应的注解

   ```java
   public class Spittle {
       private final Long id;
       @NotNull  //非空，注意：这里引入的是 javax.validation.constraints.NotNull
       @Size(min = 5,max = 16)  //字符长度为 5-16个
       private final String message;
   }
   ```

3. 在控制器的处理方法 bean 类型的前面添加`@Valid`注解

   ```java
   @RequestMapping(value = "/register",method = RequestMethod.POST)
   public String processRegistration(
       @Valid Spitter spitter,  //校验Spitter输入 
       Errors errors){
   
       if(errors.hasErrors()){
           return "registerForm";  //如果校验错误，重新返回表单
       }
       return "success";
   }
   ```

这里由于本身是使用基于注解的方式来搭建SpringMVC，所以仅三步，如果是使用基于配置文件的方式的话，还需要在SpringMVC的配置文件中添加：` <mvc:annotation-driven />`



SpringMVC 是通过对处理方法签名的规约来保存校验结果的： 将对象的校验结果保存到随后的入参中，保存校验结果的入参必须是`BindingResult`或`Errors`类型，这两个类都位于`org.springframework.validation`包中。

- 验证出错转向到定制的页面

  - 注意: 需校验的 Bean 对象和其绑定结果对象或错误对象时成对出现的，它们之间不允许声明其他的入参

    ```java
    public String handle(@Valid User user , BindingRedult userBindingResult){}
    ```

`Errors`接口提供了获取错误信息的方法 ， 如`getErrorCount()`或`getFieldErrors(String field)`。

`BindingResult`扩展了`Errors`接口。



# 3.6 返回模型数据

SpringMVC提供一下几种途径输出模型数据：

- `ModelAndView`：处理方法返回值类型为`ModelAndView`时，方法体即可通过该对象添加模型数据；
- `Map`及`Model`：入参为`org.springframework.ui.Model`、`org.springframework.ui.ModelMap`或`java.uti.Map`时，处理方法返回时，`Map`中的数据会自动添加到模型中；
- `@SessionAttributes`：将模型中的某个属性暂存到`HttpSession`中，以便多个请求之间可以共享这个属性；
- `@ModelAttribute`：方法入参标注该注解后，入参的对象会放到数据模型中。



## ModelAndView

控制器处理方法的返回值如果为`ModelAndView`，则其**既包含视图信息，也包含模型数据信息**。SpringMVC 会把 `ModelAndView`的 model 中数据放入到 request 域对象中。

```java
@RequestMapping("/testModelAndView")
public ModelAndView testModelAndView(){
    String viewName = "success";
    ModelAndView modelAndView = new ModelAndView(viewName);
    modelAndView.addObject("time",new Date());//添加模型数据到 ModelAndView 中

    return modelAndView;
}
```

可以在结果页面success.jsp中获取模型数据：

```jsp
time:${requestScope.get("time")}
```



## Map

SpringMVC 在内部使用一个`org.springframework.ui.Model`接口存储模型数据。

**SpringMVC在调用方法前会创建一个隐含的模型对象作为模型数据的存储容器**

如果**方法的入参为Map或Model类型**，SpringMVC会将隐含模型的引用传递给这些入参。在方法体内，开发者可以通过这个入参对象访问到模型中的所有数据，也可以向模型中添加新的属性数据

```java
/**目标方法可以添加 Map 类型(实际上也可以是 Model 类型或 ModelMap 类型)的参数.*/
@RequestMapping("/testMap")
public String testMap(Map<String, Object> map){
    System.out.println(map.getClass().getName()); //测试实际传入的参数类型
    map.put("names", Arrays.asList("Tom", "Jerry", "Mike"));
    return "success";
}
```

可以在结果页面success.jsp中获取模型数据：

```jsp
name:${requestScope.names}
```



## @SessionAttributes

若**希望在多个请求之间共用某个模型属性数据**，则可在控制器类上标注一个`@SessionAttributes`，SpringMVC将在模型中对应的属性暂存到 request 和 session 域对象中。

该注解除了可以**通过属性名指定**需要**放到会话中的属性**外(实际使用的是value属性值)，还可以**通过模型属性的对象类型指定哪些模型属性**需要**放到会话中**(实际使用的是types属性值)。

注意: 该注解<u>只能放在类的上面. 而不能修饰放方法</u>。

```java
@SessionAttributes(value = {"user"},types = {String.class} )
@Controller
public class RequestMappingTest {
        @RequestMapping("/testSessionAttributes")
        public String testSessionAttributes(Map<String, Object> map){
                User user = new User("Tom", "123456", "tom@atguigu.com", 15);
                map.put("user", user);
                map.put("school", "atguigu");
                return "success";
        }
}
```

可以在结果页面success.jsp中获取模型数据：

```jsp
<!--测试value-->
request user:${requestScope.user}
session user:${sessionScope.user}
<!--测试type-->
request school:${requestScope.school}
session school:${sessionScope.school}
```



## @ModelAttribute

- `@ModelAttribute`注解，只支持一个属性value，表示绑定的属性名称
  - **有`@ModelAttribute`标注的方法, 会在每个目标方法执行之前被 SpringMVC 调用！！**
  - `@ModelAttribute`注解也可以来修饰目标方法 POJO 类型的入参, 其 value 属性值有如下的作用:
    - SpringMVC会使用value属性值在`Model`中查找对应的对象, 存在则直接传入到目标方法的入参中
    - SpringMVC会以value为 key, POJO 类型的对象为value, 存入到 request 域对象中
  - 注意: `@ModelAttribute`修饰的方法中, 放入到 Map 时的键要和目标方法入参类型的第一个字母小写的字符串一致!
  - 使用：
    - 标注在有返回值的方法：则绑定的属性值会作为方法的返回值返回；
    - 标注在`void`方法上：



# 3.7 重定向/转发

一般情况下，控制器方法返回字符串类型的值会被当成逻辑视图名处理，如果返回的字符串中带**forward:** **或** **redirect:** 前缀时，SpringMVC 会对他们进行特殊处理：将 forward: 和 redirect: 当成指示符，其后的字符串作为URL 来处理。

```java
@RequestMapping("/testRedirect")
public String testRedirect(){
    System.out.println("testRedirect");
    return "redirect:/index.jsp";   //return "forward:/index.jsp";
}
```

