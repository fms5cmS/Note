# 4.监听器Listener

监听器：专门用于对其他对象身上发生的事件或状态改变进行监听和相应处理的对象，当被监视的对象发生情况时，立即采取相应的行动。

Servlet监听器：Servlet规范中定义的一种特殊类，它用于监听 web 应用程序中的`ServletContext`、`HttpSession`和` ServletRequest`等域对象的创建与销毁事件，以及监听这些域对象中的属性发生修改的事件。

- 按监听的事件类型对Servlet监听器分类：
  - 监听**域对象（pageContext、request、application）自身的创建和销毁**的事件监听器
  - 监听**域对象中的属性的增加和删除**的事件监听器
  - 监听**绑定到`HttpSession`域中的某个对象的状态**的事件监听器。<u>该监听器不用注册</u>。



## 4.0 编写监听器

Servlet 规范为每种事件监听器都定义了相应的接口，开发人员编写的事件监听器程序只需**实现**这些**接口**，Web 服务器根据用户编写的事件监听器所实现的接口把它注册到相应的被监听对象上。

**一些** Servlet事件监听器**需要在**Web应用程序的**web.xml文件中注册**，一个 web.xml文件中可以注册多个 Servlet 事件监听器，Web 服务器**按照**它们在web.xml 文件中的**注册顺序来加载和注册**这些 Serlvet 事件**监听器**。

由于一个Web 应用程序只会为每个事件监听器创建一个对象，有可能出现多个线程同时调用同一个事件监听器对象的情况，所以，在编写事件监听器类时，应考虑多线程安全的问题。

注册：

```xml
    <listener>
        <listener-class>listeners.HelloServletContextListener</listener-class>
    </listener>
```



## 4.1 监听器一

监听**域对象自身的创建和销毁**的事件监听器，即监听`ServletContext`、`HttpSession`、`HttpServletRequest`这三个对象的创建和销毁事件的监听器。

`ServletContext`、`HttpSession`、`HttpServletRequest`这三个域对象创建和销毁的时机：

| 域对象               | 创建的时机                                   | 销毁的时机                                                   |
| -------------------- | -------------------------------------------- | ------------------------------------------------------------ |
| `ServletContext`     | Web服务器启动时为每个Web应用创建相应的该对象 | Web服务器关闭时为每个Web应用销毁相应的该对象                 |
| `HttpSession`        | 浏览器开始与服务器会话时                     | 调用`HttpSession.invalidate()`；超过session的最大有效时间间隔；服务器进程被终止 |
| `HttpServletRequest` | 每次请求开始时                               | 每次访问结束后                                               |



- ==`ServletContextListener`接口==。最常用！
  - 用于监听`ServletContext`对象的创建和销毁事件。
  - `ServletContext`对象被创建后，Servlet容器立即调用`contextInitialized(ServletContextEvent sce)`方法；
  - `ServletContext`对象被销毁前（先调用方法再立刻销毁），Servlet容器调用`contextDestroyed(ServletContextEvent sce)`方法；
    - `ServletContextEvent`中的：`getServletContext()`获取`ServletContext`
  - ==可以在当前 WEB 应用被加载时对当前 WEB 应用的相关资源进行初始化操作：创建数据库连接池，创建 Spring 的 IOC 容器，读取当前 WEB 应用的初始化参数 ...==
- `HttpSessionListener`接口
  - 用于监听`HttpSession`对象的创建和销毁事件。
  - 创建一个`Session`后，Servlet容器立即调用`sessionCreated(HttpSessionEvent se)`方法；
  - 销毁一个`Session`前（先调用方法再立刻销毁），Servlet容器调用`sessionDestroyed(HttpSessionEvent se)`方法；
- `ServletRequestListener`接口
  - 用于监听`HttpServletRequest`  对象的创建和销毁事件。
  - 创建一个`ServletRequest`对象后，Servlet容器立即调用`requestInitialized(ServletRequestEvent sre)`方法；
  - 销毁一个`ServletRequest`（先调用方法再立刻销毁），Servlet容器调用`requestDestroyed(ServletRequestEvent sre)`方法



## 4.1-域对象生命周期★

==request：请求==

1. request创建：当发送一个请求时被创建；
2. request销毁：当一个响应返回时，即被销毁。
3. 注意, 请求转发的过程是一个请求对象，而重定向是两个请求对象。

==session：会话==

1. session创建：当第一次访问 WEB 应用的一个 JSP 或 Servlet 时，且该 JSP 或 Servlet 中还需要创建 session 对象。此时服务器会创建一个 session 对象；
2. session 销毁: session 过期；直接调用 session 的`HttpSession.invalidate`方法，当前Web 应用被卸载(session 可以被持久化)。
3. 注意：关闭浏览器，并不意味着 session 被销毁，还可以通过 sessionid 找到服务器中的 session 对象。

==application: （应用）贯穿于当前的 WEB 应用的生命周期==

1. application 创建：当前 WEB 应用被加载时创建 application 对象,
2. application 销毁：当前 WEB 应用被卸载时销毁 application 对象.



## 4.2 监听器二

监听**域对象中的属性的增加和删除**的事件监听器，这里的域对象就是上面的三个域对象。较少使用！

使用了`ServletContextAttributeListener`、`HttpSessionAttributeListener`、`ServletRequestAttributeListener`这三个接口分别用于监听三个域对象中属性的增、删、替换行为，每种行为在三个接口中的方法名完全相同，只是接收的参数类型不同。

- `attributeAdded`方法
  - 当向被监听器对象中增加一个属性时，Web容器就调用事件监听器的`attributeAdded`方法来响应，这个方法接受一个事件类型的参数，监听器可以通过这个参数来获得正在增加属性的域对象和被保存到域中的属性对象。
  - 对应不同域对象的完整方法定义为：
    - `public void attributeAdded(ServletContextAttributeEvent event)`
    - `public void attributeAdded(HttpSessionBindingEvent event)`
    - `public void attributeAdded(ServletRequestAttributeEvent srae)`
- `attributeRemoved`方法
  - 当删除被监听对象中的一个属性时，Web容器调用事件监听器的这个方法来响应。
  - 对应不同域对象的完整方法定义为：
    - `public void attributeRemoved(ServletContextAttributeEvent event)`
    - `public void attributeRemoved(HttpSessionBindingEvent event)`
    - `public void attributeRemoved(ServletRequestAttributeEvent srae)`
- `attributeReplaced`方法
  - 当监听器的域对象中的某个属性被替换时，Web容器调用事件监听器的这个方法来响应。
  - 对应不同域对象的完整方法定义为：
    - `public void attributeReplaced(ServletContextAttributeEvent event)`
    - `public void attributeReplaced(HttpSessionBindingEvent event)`
    - `public void attributeReplaced(ServletRequestAttributeEvent srae)`



## 4.3 监听器三

监听**绑定到`HttpSession`域中的某个对象的状态**的事件监听器。较少使用！

- 保存在 Session域中的对象可以有多种状态：
  - 绑定到  Session中；
  - 从Session域中解除绑定；
  - 随Session对象持久化到一个存储设备中；
  - 随Session对象从一个存储设备中恢复

Servlet 规范中定义了两个特殊的监听器接口来帮助 JavaBean 对象了解自己在 Session 域中的这些状态：`HttpSessionBindingListener`接口和`HttpSessionActivationListener`接口。



- `HttpSessionBindingListener`接口
  - 实现了该接口的 JavaBean 对象可以感知自己被绑定到 Session 中和从 Session 中删除的事件；
  - 当对象被绑定到`HttpSession`对象中时，Web服务器调用该对象的`public void valueBound(HttpSessionBindingEvent event) `方法
  - 当对象从`HttpSession`对象中解除绑定时，Web 服务器调用该对象的`public void valueUnbound(HttpSessionBindingEvent event)`方法



- `HttpSessionActivationListener`接口
  - 实现了该接口的 JavaBean 对象可以感知自己被活化和钝化的事件
    - 活化: 从磁盘中读取 session 对象；
    - 钝化: 向磁盘中写入 session 对象
    - session 对象存储在tomcat 服务器的 \work\Catalina\localhost\Session\SESSION.ser
  - 当绑定到`HttpSession`对象中的对象将要随`HttpSession`对象被钝化之前，Web 服务器调用该对象的`public void sessionWillPassivate(HttpSessionEvent se)`方法
  - 当绑定到`HttpSession`对象中的对象将要随`HttpSession`对象被活化之后，Web 服务器调用该对象的`public void sessionDidActivate(HttpSessionEvent se)`方法

