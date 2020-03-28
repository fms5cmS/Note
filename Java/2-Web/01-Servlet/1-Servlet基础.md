# 1.1 HelloWorld

本笔记中使用的是 Maven 工程来创建 Web 工程！

JavaServlet 是和平台无关的服务器端组件，运行在Servlet容器中。Servlet 容器负责 Servlet 和客户的通信及调用 Servlet 的方法，Servlet 和客户采用“请求/响应”的模式。 

首先通过一个 “HelloWorld” 的代码来初步认识一下。操作步骤：

- 1.导入`javax.servlet-api.jar`包（3.0以前的版本为：servlet-api.jar）

- 2.创建一个 Servlet 接口的实现类，并实现其中的方法。

  ```java
  public class HelloServlet implements Servlet{}
  ```

- 3.在 web.xml 中配置和映射 Servlet 

  - 访问路径可以是指定的字符串，如：`/hello`;也可以是一个指定的 html 文件，此时路径为 html 中的 action 属性 

```xml
<!--配置和映射 Servlet  -->
<servlet>
    <!-- Servlet注册的名字 -->
    <servlet-name>helloservlet</servlet-name>
    <!-- Servlet全类名 -->
    <servlet-class>zServlet.HelloServlet</servlet-class>
    <!-- 可以指定Servlet被创建的时机 -->
    <load-on-startup>1</load-on-startup>
</servlet>

<servlet-mapping>
    <!-- 需要和某一个servlet节点的servlet-name子节点的文本节点一致 -->
    <servlet-name>helloservlet</servlet-name>
    <!-- 映射具体的访问路径：“/”代表当前web应用的根目录http://localhost:8080/zServlet  其中 “hello”代表了访问的具体标识  -->
    <url-pattern>/hello</url-pattern>    
</servlet-mapping>
```

- 4.启动服务器 
- 5.在浏览器中输入：`http://localhost:8080/项目名/hello`



- `<servlet-mapping>`
  - 多个URL可以映射到同一个Servlet上，即一个`<servlet>`下面可以有多个`<servlet-mapping>`；
  - 在Servlet映射到的URL中也可以使用 `* `通配符，但是只能有两种固定格式：
    - `.扩展名`，eg：`<url-pattern>.html</url-pattern>`指所有的 html 文件都可以受理。
    - 以`/`开头并以`/*`结尾。eg:`<url-pattern>/*</url-pattern>` 
    - 既带` /` 又带扩展名的不合法 



- `load-on-startup`参数： 
  - 1）配置在servlet节点中   
  - 2）可以指定Servlet被创建的时机。 
    - 若为负数，则在第一次请求时被创建；
    - 若为非负数，则在当前Web应用被Servlet容器加载时创建实例，且数值越小，越早被创建。 



# 1.2 Servlet生命周期

Servlet 生命周期：

1. Servlet 通过调用 `init () `方法进行初始化。 
2. Servlet 调用 `service()`方法来处理客户端的请求。
3. Servlet 通过调用 `destroy()` 方法终止（结束）。
4. 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。 



- 由 Servlet 容器（ Web 服务器）调用的各个方法：
  - 构造器（==只调用一次==）：创建实例。 
    - 所以 Servlet 是单实例的！会有线程安全问题，不推荐在 Servlet 中写一个全局变量然后每次在`service()`中修改。 
  - `init(ServletConfig config)` 方法 ：初始化当前 Servlet
    - ==只调用一次==。在第一次创建 Servlet 时立即被调用，后续每次用户请求时不再调用。
  - `service(ServletRequest req, ServletResponse res)`方法 
    - Servlet 容器调用 `service() `处理来自浏览器的请求，并把格式化的响应写回给客户端。 
    - 每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。`service() `方法检查 HTTP 请求类型（GET、POST、PUT、DELETE 等），并在适当的时候调用 `doGet`、`doPost`、`doPut`，`doDelete` 等方法。 所以，不用对 `service() `方法做任何动作，只需要根据来自客户端的请求类型来重写 `doGet()` 或 `doPost() `即可。 
  - `destroy()` 方法 
    - 只会被调用一次，在服务器关闭，Servlet 生命周期结束时（卸载前）被调用。 



# 1.3 响应过程

Servlet容器响应客户请求的过程： 

1. Servlet引擎检查是否已经装载并创建了该Servlet的实例对象。如果是，直接执行第4步，否则，执行第2步； 
2. 装载并创建该 Servlet 的一个实例对象：调用该 Servlet 的构造器； 
3. 调用 Servlet 实例对象的`init()`方法； 
4. 创建一个用于封装请求的`ServletRequest`对象和一个代表响应消息的`ServletResponse`对象，然后调用 Servlet 的`service()`方法并将请求和响应对象作为参数传递进去；
5. Web应用程序被停止或重启前，Servlet 引擎将卸载 Servlet，并在卸载前调用 Servlet 的`destroy()`方法。 



# 1.4 Context

`init()`方法中`ServletConfig`参数可获取 Servlet 配置信息，以及用`ServletConfig`获取`ServletContext`对象后，获取Web应用程序的配置信息。 

如果表单提交的数据中有中文数据则需要转码： 

```java
String name =new String( request.getParameter("name").getBytes("ISO8859-1") , "UTF-8");
```

一个简单的 web.xml 文件（仅列出了主要部分）

```xml
	<!-- 配置当前web应用的初始化参数，该可以为所有的Servlet获取 -->
     <context-param>
         <param-name>driver</param-name>
          <param-value>com.mysql.jdbc.Driver</param-value>
     </context-param>
     
     <servlet>
         <servlet-name>helloservlet</servlet-name>
         <servlet-class>zServlet.HelloServlet</servlet-class>
         
         <!-- 配置Servlet的初始化参数，且该标签必须在<load-on-startup>前面。只能由指定的Servlet获取 -->
         <init-param>
              <param-name>user</param-name>
              <param-value>root</param-value>
         </init-param>
   
         <load-on-startup>1</load-on-startup>
     </servlet>
       
     <servlet-mapping>
         <servlet-name>helloservlet</servlet-name>
         <url-pattern>/hello</url-pattern>    
     </servlet-mapping>
```



- `ServletConfig`：封装了Servlet的配置信息，且可以获取`ServletContext`对象 

  - 获取初始化参数（使用`<init-param>`标签配置 **Servlet 的**初始化参数（见上面的web.xml））

    ```java
    public String getInitParameter(String name);//获取指定参数名的初始化参数
    public Enumeration<String> getInitParameterNames();//获取参数名组成的Enumeration对象
    
    Enumeration<String> enu = config.getInitParameterNames();
    while(enu.hasMoreElements()) {
        String name = enu.nextElement();
        String value = config.getInitParameter(name);
        System.out.println(name+":"+value);
    }
    ```

  - 获取 Servlet 的配置名称（了解，很少使用）

    ```java
    String servletName = config.getServletName();
    System.out.println(servletName);
    ```

  - 获取ServletContext对象 

    ```java
    ServletContext context = ServletConfig.getServletContext();
    ```



- ==ServletContext==

  - 调用`ServletConfig.getServletContext()`方法可以返回`ServletContext`对象的引用。

  - 该对象代表当前 web 应用，可以从中获取到当前 Web 应用程序各个方面的信息。

  - 由于==每个Web应用程序中的所有Servlet都共享同一个ServletContext对象==，所以，`ServletContext`对象被称为 application 对象（web应用程序对象）。 

  - 功能： 

    - 获取Web应用程序的初始化参数（使用`<context-param>`标签配置Web的初始化参数（见上面web.xml），该参数类似于全局变量，可以在web.xml中查看对应的位置来理解 ）

      ```java
      //用法和 ServletConfig对象的同名方法 的用法相同
      public String getInitParameter(String name);
      public Enumeration<String> getInitParameterNames();
      ```

    - 获取Web应用的某个文件在服务器上的绝对路径，而不是部署前的路径

      ```java
      public String getRealPath(String path); 
      eg：
      String path = context.getRealPath("/Animation.txt");//这里的 / 是相对于当前web应用的根目录
      ```

    - 获取当前Web应用的根目录

      ```java
      public String getContextPath();
      eg：
      String contextPath = context.getContextPath();
      ```

    - 获取当前web应用的某一文件对应的输入流 

      ```java
      public InputStream getResourceAsStream(String path);
      eg：
      InputStream is2 = context.getResourceAsStream("/WEB-INF/classes/zServlet/jdbc.properties");
      ```



# 1.5 service()方法

Servlet的`service()`方法用于应答请求：每次请求都会调用这个方法 。

```java
public void service(ServletRequest req, ServletResponse res)
//两个参数都由服务器实现，并在服务器调用 service() 方法时传入
```



## 请求

- `ServletRequest`接口：封装了请求信息，可以从中获取到任何的请求信息。

  - 子接口：`HttpServletRequest`

  - 获取请求参数

    ```java
    /** 根据请求参数的名字，返回参数。若请求参数由多个值（如多选框），只能获得第一个值 */
    public String getParameter(String name)
    /**根据请求参数的名字，返回请求参数对应的字符串数组。 多选框*/
    public String[] getParameterValues(String name)
    /**返回参数名对应的Enumeration对象*/ 
    public Enumeration<String> getParameterNames()
    /**返回请求参数的键值对，key：参数名；value：参数值（String数组类型）*/
    public Map<String, String[]> getParameterMap()
    ```

    ```java
    ServletRequestString user = req.getParameter("user"); 
    //req是service()方法的ServletRequest参数
    
    String[] interestings = req.getParameterValues("interesting");
    System.out.println(Arrays.toString(interestings));
    
    Enumeration<String> enu = req.getParameterNames();
    while(enu.hasMoreElements())
        System.out.print(enu.nextElement()+" ");
    System.out.println();
    
    Map<String,String[]> map = req.getParameterMap();
    Set<String> keys = map.keySet();
    for(String key:keys) {
        System.out.println(key+"-->"+Arrays.toString(map.get(key)));
    }
    ```

  - 获取请求的URI（见下面）

  - 获取请求方式（见下面）

  - 若是一个GET请求，获取请求参数对应的字符串。即 ？ 后的字符串；如果是POST请求，返回为null   

  - 获取请求的Servlet的映射路径

    ```java
    HttpServletRequest httpreq = (HttpServletRequest)req;    //需要进行强制类型转换
    //HttpServletRequest是ServletRequest的子接口，针对HTTP请求所定义，包含了获取HTTP请求的相关方法
    
    //如果URL是：http://localhost:8080/zServlet/loginServlet?user=xsdf&password=sf&interesting=Animation&interesting=Mooc
    //获取请求的URI，在站点信息后面 /zServlet/loginServlet
    String requestURI = httpreq.getRequestURI(); 
    
    String method = httpreq.getMethod();    //获取请求方式 GET
    
    //获取URL中的参数。user=xsdf&password=sf&interesting=Animation&interesting=Mooc
    String queryString = httpreq.getQueryString();    
    //获取请求的Servlet的映射路径 /loginServlet
    String servletPath = httpreq.getServletPath();   
    ```



## 响应

- `ServletResponse`接口：封装了相应信息，如果想给用户什么响应，具体可以使用该接口的方法实现。

  - 将信息打印到浏览器

    ```java
    public PrintWriter getWriter()：//返回PrintWriter对象
    eg：
    PrintWriter out = res.getWriter(); //res是service()方法的ServletResponse参数
    out.println("hello world");//把print()中的参数打印到客户的浏览器上。
    ```

  - 设置响应内容的类型

    ```java
    public void setContentType(String type)：//设置响应的内容类型
    //注意：参数中，不同类型对应的字符串可以在Tomcat安装路径下的config文件夹中的web.xml中查找
    eg：
    res.setContentType("application/msword"); //设置响应类型为word文档，下面的hello world 会在一个word文档中输出。
    PrintWriter out = res.getWriter();
    out.println("hello world");
    ```

    ```java
    public ServletOutputStream getOutputStream()：//获取一个输出流，如果想要下载一个文档的话会使用到
    ```



# 1.6  HttpServlet

在最开始 “HelloWorld” 中，我们是通过实现 Servlet 接口来创建的，但 Servlet 接口中有很多的空方法，不利于实际开发，为了简化开发，我们可以继承 `GenericServlet`抽象类。

`GenericServlet`（了解）是实现了`Servlet`、`ServletConfig`、`Serializable`的抽象类。其中`service()`方法为抽象方法。 我们只需要实现这个方法即可。

1. `GenericServlet`中有一个`ServletConfig`类型的成员变量config，在`init(ServletConfig config)`中对其初始化，并利用config实现了`ServletConfig`接口的方法。 
2. 除了`init(ServletConfig config)`，还有一个无参数的`init()`方法。覆盖时如果覆盖了带参的`init()`方法，就无法为成员变量config初始化，使用时会出现空指针异常，所以它提供了一个空的`init()`方法用于用户覆盖。 如果要覆盖`init(ServletConfig config)`，注意添加语句：`super.init(config); `
3. `init（）`方法并不是重写了Servlet的方法，即不是Servlet生命周期的方法，不是由服务器直接调用，而是在`init（ServletConfig config）`中对其调用。 



尽管实现`GenericServlet`接口来创建 Servlet 已经很方便了，但是在获取请求方式时，还必须先将`ServletRequest`类型的 request 转型为`HttpServletRequest`类型，然后才能`getMethod()`，为了更进一步简化开发： 我们可以==继承`HttpServlet`抽象类==。

`HttpServlet`是继承了`GenericServlet`的抽象类，针对于HTTP协议定制。 

1. `HttpServlet`在`service(ServletRequest req, ServletResponse res)`中将`ServletRequest`、`ServletResponse`类型强转为`HttpServletRequest`、`HttpServletResponse`类型，使用处理异常的方式使代码更简洁，并调用重载的`service(HttpServletRequest req, HttpServletResponse resp)`：

   ```java
   public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException{
       HttpServletRequest  request;
       HttpServletResponse response;
   
       if (!(req instanceof HttpServletRequest && res instanceof HttpServletResponse)) {
           throw new ServletException("non-HTTP request or response");
       }
       request = (HttpServletRequest) req;
       response = (HttpServletResponse) res;
   
       service(request, response);
   }
   ```

2. 在`service(HttpServletRequest req, HttpServletResponse resp)`中获取了请求方式，跟据请求方式Xxx，创建了doXxx方法为具体实现的方法； 所以，==实际开发中，直接继承HttpServlet，根据请求方式覆盖doXxx()方法。==



# 1.7 请求重定向&转发

- `RequestDispatcher`接口，用`forward()`方法实现==请求转发==

  ```java
  //测试请求和重定向中，请求的request和中转的request是否为同一对象
  //request.setAttribute("name", "abcdef");
  //System.out.println("ForwardServlet's name:"+request.getAttribute("name"));
  
  //请求的转发
  //1.调用HttpServletRequest的getRequestDispatcher()方法获取RequestDispatcher对象
  //调用getRequestDispatcher()需要传入要转发的地址
  String path = "testServlet";
  RequestDispatcher requestDispatcher = request.getRequestDispatcher("/"+path);//"/"表示当前应用的根目录
                 
  //2.调用HttpServletRequest的forward(request,response)进行请求的转发
  requestDispatcher.forward(request, response);
  ```

- `sendRedirect()`方法实现==请求重定向==

  ```java
  //执行请求的重定向,直接调用response的sendRedirect(path)方法    
  //path为要重定向的地址
  String path = "testServlet";
  response.sendRedirect(path);
  ```



- 请求重定向和转发的**本质区别**：
  - 请求的转发客户端只发出了一次请求，而重定向则发出了两次请求。 
    - A向B借钱，B说我给你解决，然后B向C借了钱给了A，A只发出一次请求；（转发）
    - A向B借钱，B说我没有，你去向C借吧，于是A向C借钱，A发出了两次请求。（重定向）
- ==**具体的区别**==：
  - 区别一：
    - 请求的转发：地址栏是初次发出请求的地址。
    - 请求的重定向：地址栏为最后响应的地址。 
  - 区别二：
    - 请求的转发：在最终的Servlet中，request对象和中转的那个request是同一个对象。
    - 请求的重定向：在最终的Servlet中，request对象和中转的那个request不是同一个对象。
  - 区别三：
    - 请求的转发：只能转发给当前Web应用的资源。
    - 请求的重定向：可以重定向到任何资源。
  - 区别四：
    - 请求的转发：`/`代表的是当前Web应用的根目录。
    - 请求的重定向：`/`代表的是当前Web站点的根目录 
- 什么时候转发，什么时候重定向？ 
  - 答：若目标的响应页面不需要从`request`中读取任何值，可以使用重定向。（还可以防止表单的重复提交） 

