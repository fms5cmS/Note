# 4.0 JSP行为

JSP 提供了一种称之为 Action 的元素，在 JSP 页面中使用 Action 元素可以完成各种通用的 JSP 页面功能，也可以实现一些处理复杂业务逻辑的专用功能。 

每个 Action 元素在 JSP 页面中都以 XML 标签的形式出现，用来控制servlet引擎。它能够动态插入一个文件，重用JavaBean组件，引导用户去另一个页面，为Java插件产生相关的HTML等等。

JSP规范中定义了一些标准的 Action 元素，这些元素的标签名都以 jsp 作为前缀，且全部小写，如：`<jsp:include>`、`<jsp:forward>`等等。 下面列举了一些可用的 JSP 行为标签：

| **语法**        | **描述**                                                   |
| --------------- | ---------------------------------------------------------- |
| jsp:include     | 用于在当前页面中包含静态或动态资源                         |
| jsp:useBean     | 寻找和初始化一个JavaBean组件                               |
| jsp:setProperty | 设置 JavaBean组件的值                                      |
| jsp:getProperty | 将 JavaBean组件的值插入到 output中                         |
| jsp:forward     | 从一个JSP文件向另一个文件传递一个包含用户请求的request对象 |
| jsp:plugin      | 用于在生成的HTML页面中包含Applet和JavaBean对象             |
| jsp:element     | 动态创建一个XML元素                                        |
| jsp:attribute   | 定义动态创建的XML元素的属性                                |
| jsp:body        | 定义动态创建的XML元素的主体                                |
| jsp:text        | 用于封装模板数据                                           |

没有详细说明的行为标签，可以查看[阿里云大学-JSP自学手册](https://edu.aliyun.com/lesson_503_5646?spm=5176.10731542.0.0.g8lesa&accounttraceid=34b865be-2cd3-4b55-a004-ee891c4e9e56#_5646)

- 常见的属性
  - id 属性：是动作元素的唯一标识，可以在JSP页面中引用。动作元素创建的id值可以通过PageContext来调用。 
  - scope 属性：用于识别动作元素的生命周期。 id 属性和 scope 属性有直接关系，scope属性定义了相关联 id 对象的寿命。 scope属性有四个可能的值： (a) `page`, (b)`request`, (c)`session`, 和 (d) `application`。 



# 4.1 `<jsp:include>`标签

用于把另一个资源的输出内容插入进当前 JSP 页面的输出内容中，这种在 JSP 页面执行时的插入方式称之为==动态引入==。

- 语法：

  ```jsp
  <jsp:include page="文件地址" flush="true"></jsp:include>     
  <!-- page 属性指包含在页面中的相对URL地址；flush 属性定义在包含资源前是否刷新缓存区 -->
  ```

  动态引入：不是像include指令生成一个Servlet源文件，而是两个，然后通过一个方法的方式把目标页面包含进来的。 

- include 指令和 `<jsp:include>`标签

  - `<jsp:include>`标签是在当前 JSP 页面的执行期间插入被引入资源的输出内容，当前 JSP 页面与被动态引入的资源是两个彼此独立的执行实体，被动态引入的资源必须是一个能独立被 Web 容器调用和执行的资源。插入文件的时间是在页面被请求的时候。 
  - include 指令在JSP文件被转换成Servlet的时候引入遵循 JSP 格式的文件 ，被引入文件与当前 JSP 文件共同被翻译成一个 Servlet源文件。



# 4.2 `<jsp:forward>`标签

用于把请求转发给另一个资源。

- 语法：

  ```jsp
  <jsp:forward page="/include/b.jsp"></jsp:forward>
  <!--相当于：-->
  <%
      request.getRequestDispatcher("/include/b.jsp").forward(request, response);
  %>
  ```

- 使用 `<jsp:forward>` 可以使用`<jsp:param>`子标签向 b.jsp 传入一些参数，同样 `<jsp:include>` 也可以使用`<jsp:param>`子标签。参数可以在转发后或包含的页面中通过`request.getParameter("xxx")`获取。

  ```jsp
  <!-- 可以在转发后页面获取 -->
  <jsp:forward page="/include/b.jsp">
  	<jsp:param value="abcd" name="username"/>
  </jsp:forward>
  ```

  ```jsp
  <jsp:include page="/include/b.jsp">
  	<jsp:param value="abcd" name="username"/>
  </jsp:include>
  ```

