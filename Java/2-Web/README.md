_说明：视频笔记，视频：尚硅谷 Web 教程_

# 0.1 静/动态网页

判断网页是静态还是动态的，观察其是否随着时间、地点、用户操作的改变而改变。

动态网页需要使用到服务端脚本语言，如 JSP。

# 0.2 架构

- CS：Client Server
  - 优点：
    - 性能较高：可以将一部分的计算工作放在客户端上,这样服务器只需要处理数据即可。
    - 界面酷炫:客户端可以使用更多系统提供的效果,做出更为炫目的效果。
  - 缺点：
    - 更新软件：如果有新的功能，就要推出新的版本。
    - 不同设备访问：如果使用其他的电脑，没有安装客户端的话就无法登陆软件。
- BS：Browser Server 客户端可通过浏览器直接访问服务端
  - 优点：
    - 更新简洁：如果需要更新内容了,对开发人员而言需要更改服务器的内容，对用户而言只需要刷新浏览器即可。
    - 多设备同步：所有数据都在网上,只要能够使用浏览器即可登录使用。
  - 缺点：
    - 性能较低：相比于客户端应用性能较低,但是随着硬件性能的提升,这个差距在缩小。
    - 浏览器兼容：处理低版本的浏览器显示问题一直是前端开发人员头痛的问题之一。移动设备兼容性较好，ie6 已经越来越少人用了。

# 0.3 HTTP 请求

- HTTP 请求消息：
  - 请求消息的结构：
    - ==一个请求行、若干消息头、以及实体内容==
    - 其中的一些消息头和实体内容都是可选的，==消息头和实体内容之间要用空行隔开==。
  - 响应消息的结构：
    - ==一个状态行、若干消息头、以及实体内容==
    - 其中的一些消息头和实体内容都是可选的，==消息头和实体内容之间要用空行隔开==。

* GET 请求方式

  - 什么样的请求方式一定是 GET 方式？

    - ① 在浏览器地址栏中输入某个 URL 地址或单击网页上的超链接时，浏览器发出的 HTTP 请求消息的请求方式为 GET 方式；
    - ② 如果网页中的`<form>`表单元素的`method`属性被设置为了“GET”，浏览器提交这个 FORM 表单时生成的 HTTP 请求消息的请求方式为 GET 方式；

  - GET 请求方式将请求的编码信息添加在网址后面，网址与编码信息通过"?"号分隔。如：

    ```
    http://www.test.com/hello?key1=value1&key2=value2  //直接将参数附在URL后面
    ```

  - 使用 GET 方式传送的数据量一般限制在 1KB 以下。

* POST 请求方式

  - POST 请求方式主要用于向 Web 服务器端程序提交 FORM 表单中的数据：form 的 method 设置为 POST；
  - POST 方式将各个表单字段元素及其数据作为 HTTP 消息的实体内容发送给 Web 服务器，传送的数据量要比 GET 方式大得多；
  - POST 提交数据是不可见的，GET 是通过在 url 里面传递的。

# 0.4 问题

- 创建 Maven 的 Web 工程时，出现`web.xml is missing and <failOnMissingWebXml> is set to true`错误，可以在 pom.xml 文件中加入以下代码：

  ```xml
  <properties>
      <failOnMissingWebXml>false</failOnMissingWebXml>
  </properties>
  ```

  或

  ```xml
      <build>
          <plugins>
              <plugin>
                  <groupId>org.apache.maven.plugins</groupId>
                  <artifactId>maven-war-plugin</artifactId>
                  <version>3.2.0</version>
                  <configuration>
                      <failOnMissingWebXml>false</failOnMissingWebXml>
                  </configuration>
              </plugin>
          </plugins>
      </build>
  ```

# 0.5 路径

- 绝对路径：
  - 本地绝对路径：增加盘符的路径，如`e:/test/test.html`；
  - 网络绝对路径：增加协议，IP 地址，端口号的路径，如：`http://localhost:8080/test/test.html`
- 相对路径：以**当前资源的访问路径为基准**，查找其他路径。
- 路径以`/`开头，表示特殊的相对路径，不同场景中，相对位置不同：
  - 前台(由浏览器解析)：`/`是**相对服务器**的根
    - 如：超链接、表单中的 action、请求重定向
    - `<a href="/sss"></a>`由浏览器解析后为：`http://IP地址:端口号/sss`
  - 后台(由服务器解析)：`/`是**相对当前 Web 应用**的根
    - 如：亲贵转发、web.xml 中映射 Servlet 访问路径
    - `forward("/user.jsp")`由服务器解析后为：`http://IP地址:端口号/项目名/user.jsp`

# 0.6 映射

`/` 表示拦截所有请求(包括静态资源)，但不包括 \*.jsp；

`/*` 表示拦截所有请求(包括静态资源、\*.jsp)。jsp 页面是 Tomcat 的 jsp 引擎解析的；

`/**` 表示拦截任意多层路径。
