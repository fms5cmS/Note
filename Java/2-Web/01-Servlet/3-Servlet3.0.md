# 3.1 Servlet注解

Servlet 3.0 以后提供了注解(annotation)，使得不再需要在web.xml文件中进行Servlet的部署描述。完成了一个使用注解描述的Servlet程序开发。 需要运行在Tomcat7.0以上的版本。

使用`@WebServlet`将一个继承于`javax.servlet.http.HttpServlet`的类定义为Servlet组件。 

`@WebServlet`有很多的属性： 

| NO.  | 属性名          | 描述                                       |
| ---- | --------------- | ------------------------------------------ |
| 1    | asyncSupported  | 声明Servlet是否支持异步操作模式，默认false |
| 2    | description     | Servlet的描述                              |
| 3    | displayName     | Servlet的显示名称                          |
| 4    | initParams [ ]  | Servlet的init参数                          |
| 5    | name            | Servlet的名称                              |
| 6    | urlPatterns [ ] | Servlet的访问URL                           |
| 7    | value [ ]       | Servlet的访问URL                           |

Servlet 的访问URL是 Servlet 的必选属性，可以选择使用`urlPatterns`或者`value`定义。 

```java
/**
 * 注解WebServlet用来描述一个Servlet
 * 属性name描述Servlet的名字,可选
 * 属性urlPatterns定义访问的URL,或者使用属性value定义访问的URL.(定义访问的URL是必选属性)
 */
@WebServlet(name="Servlet3Demo",urlPatterns="/Servlet3Demo")
public class Servlet3Demo extends HttpServlet {
   
    private static final long serialVersionUID = 1L;
    public void doGet(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        response.getWriter().write("Hello Servlet3.0");
    }
    public void doPost(HttpServletRequest request, HttpServletResponse response)
            throws ServletException, IOException {
        this.doGet(request, response);
    }
}
```

类似的，如果要注册过滤器就使用`@WebFilter`，监听器`@WebListener`......不过原生Servlet使用较少，所以不常用。



# 3.2 Pluggability

Shared libraries / runtimes pluggability：共享库及运行时插件能力。

1. 提供`ServletContainerInitializer`的实现类
2. Servlet容器启动时会扫描当前应用里**每一个 jar包**内在 META-INF/services/ 目录下的名为javax.servlet.ServletContainerInitialize的文件(文件的内容就是`ServletContainerInitializer`实现类的全类名)
3. 加载这个文件指定的`ServletContainerInitializer`的实现类。

```java
/**
 * 使用@HandlesTypes传入感兴趣的类
 * 容器启动时，会将@HandlesTypes指定的这个类下的子类(实现类、子接口)传递过来
 * @author +1day
 */
@HandlesTypes(value = {HelloServlet.class})
public class MyServletContainerInitializer implements ServletContainerInitializer {
    /**
     * 应用启动时，会运行该方法
     * @param c 感兴趣的类的子类型
     * @param ctx 代表当前Web应用的ServletContext，使用它来注册Web的三大组件(Servlet、Filter、Listener)
     * @throws ServletException
     */
    @Override
    public void onStartup(Set<Class<?>> c, ServletContext ctx) throws ServletException {
        //注册监听器
        ctx.addListener(HelloListener1.class);

        //注册过滤器
        FilterRegistration.Dynamic filter = ctx.addFilter("helloFilter", HelloFilter.class);
        filter.addMappingForUrlPatterns(EnumSet.of(DispatcherType.FORWARD),true,"/*");

        //注册Servlet
        ServletRegistration.Dynamic servlet = ctx.addServlet("testServlet", TestServlet.class);
        servlet.addMapping("/test");

    }
}
```

- 使用编码的方式，在项目启动时给ServletContext添加组件(必须在项目启动时添加)有两种方式：
  1. 使用`ServletContainerInitializer`的`onStartup()`方法的`ServletContext`参数来添加；
  2. 使用`ServletContextListener`的`contextInitialized()`方法的`ServletContextEvent`参数先得到`ServletContext`再添加。

SpringMVC中用到了该特性！！！





# 3.3 异步请求处理

Servlet 3.0以前，Servlet采用Thread-Per-Request方式处理请求：每一次Http请求都由某一个线程从头到尾负责处理。

如果一个请求需要进行IO操作，比如访问数据库、调用第三方服务接口等，那么其所对应的线程将同步地等待IO操作完成， 而IO操作是非常慢的，所以此时的线程并不能及时地释放回线程池以供后续使用，在并发量越来越大的情况下，这将带来严重的性能问题。即便是像Spring、Struts这样的高层框架也脱离不了这样的桎梏，因为他们都是建立在Servlet之上的。为了解决这样的问题，Servlet 3.0引入了异步处理，然后在Servlet 3.1中又引入了非阻塞IO来进一步增强异步处理的性能。

```java
@WebServlet(value = "/async",asyncSupported = true)
public class HelloAsyncServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        System.out.println("主线程开始..."+Thread.currentThread().getName()+"==>"+System.currentTimeMillis());
        //1.支持异步处理:在@WebServlet上设置asyncSupported = true
        //2.开启异步模式
        AsyncContext asyncContext1 = req.startAsync();

        //3.业务逻辑进行异步处理，开始异步处理
        asyncContext1.start(()-> {
            try {
                System.out.println("副线程开始..."+Thread.currentThread().getName()+"==>"+System.currentTimeMillis());
                sayHello();
                //表明异步处理完成
                asyncContext1.complete();
                //获取异步的Context对象
                AsyncContext asyncContext2 = req.getAsyncContext();
                //4.获取响应
                ServletResponse response = asyncContext2.getResponse();
                response.getWriter().write("hello async");
                System.out.println("副线程结束..."+Thread.currentThread().getName()+"==>"+System.currentTimeMillis());
            } catch (InterruptedException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
        });
        System.out.println("主线程结束..."+Thread.currentThread().getName()+"==>"+System.currentTimeMillis());
    }

    /**模拟一个很耗时的操作*/
    public void sayHello() throws InterruptedException {
        System.out.println(Thread.currentThread().getName()+" processing...."+"==>"+System.currentTimeMillis());
        Thread.sleep(3000);
    }
}
```
