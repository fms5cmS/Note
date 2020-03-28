# 2.0 重要

## 路径

- 绝对路径：
  - 本地绝对路径：增加盘符的路径，如`e:/test/test.html`；
  - 网络绝对路径：增加协议，IP地址，端口号的路径，如：`http://localhost:8080/test/test.html`
- 相对路径：以**当前资源的访问路径为基准**，查找其他路径。
- 路径以`/`开头，表示特殊的相对路径，不同场景中，相对位置不同：
  - 前台(由浏览器解析)：`/`是**相对服务器**的根
    - 如：超链接、表单中的action、请求重定向
    - `<a href="/sss"></a>`由浏览器解析后为：`http://IP地址:端口号/sss`
  - 后台(由服务器解析)：`/`是**相对当前Web应用**的根
    - 如：亲贵转发、web.xml中映射Servlet访问路径
    - `forward("/user.jsp")`由服务器解析后为：`http://IP地址:端口号/项目名/user.jsp`



## 映射

`/`   表示拦截所有请求(包括静态资源)，但不包括 *.jsp；

`/*` 表示拦截所有请求(包括静态资源、*.jsp)。jsp页面是Tomcat的 jsp 引擎解析的；

`/**` 表示拦截任意多层路径。



## SSM中的乱码

- POST方式时的乱码问题：增加`CharacterEncodingFilter`过滤器

- GET方式时的乱码问题：在Tomcat配置文件server.xml的`<Connector port="8080"..>`标签中加入`URIEncoding="UTF-8"`

  ```xml
  <Connector port="8080" protocol="HTTP/1.1" connectionTimeout="20000"
             redirectPort="8443" useBodyEncodingforURI="true"/>
  ```



# 2.1 登录账号保存

Web中要保存/共享数据有四个范围：页面(PageContext)、请求(Request)、会话(Session)、应用(ServletContext即application)，这四个范围的大小：应用 > 会话 > 请求 > 页面。

- 账号保存位置选择：
  - 请求范围要跨越多个页面，所以使用PageContext不行；
  - 由于使用Ajax，不会跳转页面，如果保存在request中，下一个页面无法获取，所以使用request不行；
  - application对象只有一个，如果使用application，后保存的会将前面的覆盖；
  - 所以使用Session来保存登录账号的数据。
    - 如果Session中有多个数据的话一个个移除过于麻烦，可直接让Session失效`session.invalidate()`



# 2.2 Servlet

Servlet（Server Applet），全称Java Servlet，是**用Java编写的服务器端程序**。Servlet指任何实现了`Servlet`接口的类。

主要功能在于交互式地**浏览和修改**数据，**生成动态Web内容**。运行于支持Java地应用服务器中。



- Servlet的生命周期：
  - 构造器（==只调用一次==）：创建实例；
  - Servlet 通过调用 `init () `方法进行初始化；
    - ==只调用一次==。在第一次创建 Servlet 时立即被调用，后续每次用户请求时不再调用
  - Servlet 调用 `service()`方法来处理客户端的请求；
    - 每次服务器接收到一个 Servlet 请求时，服务器会产生一个新的线程并调用服务。
  - Servlet 通过调用 `destroy()` 方法终止（结束）；
    - 只会被调用一次，在服务器关闭，Servlet 生命周期结束时（卸载前）被调用
  - 最后，Servlet 是由 JVM 的垃圾回收器进行垃圾回收的。 



# 2.3JSP和Servlet

JSP（Java Server Pages）是简化Servlet编写的一种技术。将 Java 代码和 HTML 语句混合在同一个文件(.jsp)中编写；

Servlet的应用逻辑只在Java文件中，与HTML分离。

JSP侧重视图，Servlet主要用于控制逻辑。



# 2.4 JSP内置对象

|  内置对象   | 描述                                                         |
| :---------: | ------------------------------------------------------------ |
|   request   | **HttpServletRequest类的实例**                               |
|  response   | HttpServletResponse类的实例。在JSP页面中几乎不会调用response的任何方法 |
|     out     | **PrintWriter类的实例，用于把结果输出到网页上**              |
|   session   | **HttpSession类的实例，代表浏览器和服务器的一次会话。**      |
| application | **ServletContext类的实例，代表当前Web应用**                  |
|   config    | ServletConfig类的实例，若要访问当前JSP配置的初始化参数，需要通过映射的地址才可以 |
| pageContext | **PageContext类的实例，网页的属性在这里管理**                |
|    page     | 指向当前JSP对应的Servlet对象的引用，但为Object类型，只能调用Object类的方法 |
|  exception  | **Exception类的对象，代表发生错误的JSP页面中对应的异常对象** |

- 四大作用域：
  - pageContext
  - request
  - session
  - application



# 2.5 HTML、CSS、JS、jQUery

- HTML定义网页的结构；
- CSS美化页面；
- JS主要用于验证表单，做动态交互(Ajax)

jQuery是一个JS框架，封装了JS的属性和方法，用户使用更便利；并增强了JS的功能。

jQuery中的Ajax也是通过原生的JS封装的。如果采用原生JS实现Ajax非常麻烦，且每次都是一样的，也需要自己封装对象的方法和属性



# 2.6 jQuery的页面加载完毕事件

很多时候必须等到元素被加载完成后再能获取该元素。一般获取元素做操作都要在页面加载完毕后操作。

jQuery的入口函数有三种：

```javascript
//文档加载完毕，图片不加载的时候，就可以执行这个函数。
$(document).ready(function(){
    //...
});
//$(document)把原生document对象转为jQuery对象，转换完成后才能调用ready方法
//ready(function(){})表示页面结构被加载完成后执行传入的函数
```

```javascript
//文档加载完毕，图片不加载的时候，就可以执行这个函数。
$(function(){
    //...
});
//页面加载完成后执行传入的函数
```

```javascript
//文档加载完毕，图片也加载完毕的时候，在执行这个函数。
$(window).ready(function () {
    //...
})
```



- jQuery页面加载完毕和`window.onload`的区别

  - jQuery中的页面加载完毕事件，表示的是页面结构被加载完毕；

  - `window.onload`表示的是页面(包括图片、声音、图像等远程资源)被加载完毕

    ```javascript
    //不仅要等文本加载完毕，而且要等图片也要加载完毕，才执行函数。
    window.onload = function () {
        alert(1);
    }
    ```



# 2.7 Ajax

Ajax：Asynchronous Javascript And XML（异步 JavaScript 和 XML）。

作用：通过Ajax与服务器进行数据交换，可以使网页实现局部更新（在不重新加载整个网页的情况下，对网页的某部分进行更新）。

不使用Ajax的话，每次UI线程项服务器发出请求时都会先将页面内容清空(页面会变白)，等到服务器响应以后再根据响应内容对页面渲染，这样就会导致页面的闪烁现象，而使用Ajax时，UI线程会再开启一个Ajax线程来法宠请求，这样就不会发生闪烁现象。

核心是一个JS对象：**XMLHttpRequest**

- 发送Ajax请求的五个步骤：
  1. 创建异步对象。即 `XMLHttpRequest` 对象。
  2. 使用open方法设置请求的参数。`open(method, url, async)`。参数：请求的方法、请求的url、是否异步。
  3. 发送请求。
  4. 注册事件。 注册`onreadystatechange`事件，状态改变时就会调用。
  5. 获取返回的数据。

使用场景：登录失败时不跳转页面；注册时提示用户名是否存在等。



# 2.8 H5和CSS3

HTML5在HTML4基础上增强了一些标签，增加了一些如画板、声音、视频、web存储等高级功能。在H5中无论头部、主题、导航等模块需要使用使用不同标签来表示。

CSS3在CSS2基础上，增强或新增了许多特性，提供一些原理CSS2中实现起来困难或不能实现的功能：盒子边框、盒子和文字的阴影、渐变、转换、移动、缩放、旋转等、过渡、动画、可使用媒体查询实现响应式网站。

CSS3最大缺点是要根据不同浏览器处理兼容性，对应有一些处理兼容性的工具。

