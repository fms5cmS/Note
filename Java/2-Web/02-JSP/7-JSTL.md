# 7.0 JSTL！！！！

JSTL（JavaServer Pages Standard Tag Library）JSP 标准标签函数库，用于取代传统直接在页面上嵌入 Java 程序
（Scripting）的做法，以提高程序可读性、维护性和方便性。

JSTL提供了五大类标签函数库：

| JSTL标签库                                            | 前缀 | URI                                    | 范例               |
| ----------------------------------------------------- | ---- | -------------------------------------- | ------------------ |
| ==核心标签库 (Core tag library)==                     | c    | http://java.sun.com/jsp/jstl/core      | `<c:out>`          |
| I18N 格式标签库 (I18N-capable formatting tag library) | fmt  | http://java.sun.com/jsp/jstl/fmt       | `<fmt:formatDate>` |
| SQL 标签库 (SQL tag library)                          | sql  | http://java.sun.com/jsp/jstl/sql       | `<sql:query>`      |
| XML 标签库 (XML tag library)                          | x    | http://java.sun.com/jsp/jstl/xml       | `<x:forBach>`      |
| 函数标签库 (Functions tag library)                    | fn   | http://java.sun.com/jsp/jstl/functions | `<fn:split>`       |

JSTl也支持El的语法。

使用前必须引入两个jar包：

```xml
        <dependency>
            <groupId>javax.servlet</groupId>
            <artifactId>jstl</artifactId>
            <version>1.2</version>
        </dependency>
        <dependency>
            <groupId>taglibs</groupId>
            <artifactId>standard</artifactId>
            <version>1.1.2</version>
        </dependency>
```



# 7.1 核心标签库

引入标签

```jsp
<%@ taglib prefix="c" uri="http://java.sun.com/jsp/jstl/core" %>
```



| 功能分类   | 标签名称                                                   |
| ---------- | ---------------------------------------------------------- |
| 表达式操作 | **`out`**、**`set`**、`remove`、`catch`                    |
| 流程控制   | **`if`**、`choose`、`when`、`otherwise`                    |
| 迭代控制   | **`forEach`**、`forTokens`                                 |
| URL操作    | `import`、`param`、**`url`**、`param`、`redirect`、`param` |

标签中的属性如果不特殊说明都支持EL表达式。



## 表达式操作

- `<c:out>`用于显示数据的内容。三个属性：
  - `value`：需要显示的值；
  - `default`：如果`value=null`，则显示`default`的值。如果没有设置`default`，则显示一个空字符串
  - `escapeXml`：是否转换特殊字符（如：`<`转换为`&lt`），默认值为`true`。几乎不会使用该属性

```jsp
<c:out value="<java>" default="无"/>
```



- `<c:set>`用于将变量存储到JSP范围中或JavaBean的属性中。五个属性：
  - `value`：要被存储的值
  - `var`：欲存入的变量名。不支持EL表达式
  - `scope`：`var`变量的JSP范围，默认值为`page`。不支持EL表达式
  - `target`：传入的JavaBean或`java.util.Map`对象
  - `property`：指定`target`对象的属性名

```jsp
<c:set var="name" value="bx" scope="page"/>
<!--等价于：-->  <% pageContext.setAttribute("name","bx");%>
name:${pageScope.name}
```

```jsp
    <%
        Customer customer = new Customer();
        customer.setAge(17);
        request.setAttribute("customer",customer);
    %>
	age：${requestScope.customer.age}<br/>    输出为17
    <c:set target="${requestScope.customer}" property="age" value="${param.age}"/>
<!--测试网址：http://localhost:8080/jstl/jstl.jsp?age=23-->
    age:${requestScope.customer.age}<br/>      输出为23
```



- `<c:remove>`用于移除变量。较少使用。两个属性：
  - `var`：欲移除的变量名。不支持EL表达式
  - `scope`：`var`变量的JSP范围，默认为`page`。不支持EL表达式

```jsp
    <c:set var="l" value="bx" scope="page"/>
    pre:${pageScope.l}<br/>     输出：pre:bx
    <c:remove var="l" scope="page"/>
    next:${pageScope.l}<br/>	输出：next:
```



- `<c:catch>`用于处理产生错误的异常状况，并将错误信息储存起来。属性：
  - `var`：用于储存错误信息的变量名。不支持EL表达式

```jsp
<c:catch var="message">
	//可能发生错误的部分,发生错误后，这部分程序会被终止忽略，但整个网页不会中止
</c:catch>
```





## 流程控制

- `<c:if>`。三个属性：
  - `test`：若表达式结果为true，执行标签体内容，false则不执行。
  - `var`：存储`test`运算后的结果，即 true/false。不支持EL表达式
  - `scope`：`var`变量的JSP范围，默认`page`。不支持EL表达式

```jsp
    <c:if test="${pageScope.name=='bx'}">
        <c:out value="阿姨洗铁路"/> 
    </c:if>
```



- `<c:choose>`、`<c:when>`、`<c:otherwise>`这三个标签一起使用
  - 在同一个`<c:choose>`中，`<c:when>`必须在`<c:otherwise>`之前，`<c:otherwise>`必须是最后一个标签
  - 在同一个`<c:choose>`中，当有好几个`<c:when>`都符合条件时，只能有一个`<c:when>`成立

```jsp
<c:choose>
	<c:when test="${param.age > 24}">大学毕业</c:when>
	<c:when test="${param.age > 20}">高中毕业</c:when>
	<c:otherwise>高中以下...</c:otherwise>
</c:choose>
```



## 迭代控制

- `<c:forEach>`为循环控制，可将**集合(包含数组、Map)**中的成员遍历。常用！属性：
  - `var`：存放当前指向的成员。不支持EL
  - `items`：被迭代的集合对象
  - `varStatus`：用于存放当前指向的成员信息的变量名。不支持EL。也有属性：
    - `index`：当前成员的索引；
    - `count`：指到的成员的总数
    - `first`：当前成员是否为第一个成员
    - `last`：当前成员是否为最后一个成员
  - `begin`：开始的位置，默认值为0。若`begin`>`end`会产生空的结果。
  - `end`：结束的位置，默认值为最后一个成员
  - `step`：每次迭代的间隔数，默认值为1。

```jsp
    <c:forEach items="${requestScope.customers}" var="customer">
        ${customer.id}~~~~${customer.name}~~~${customer.age} 
    </c:forEach>
```

```jsp
    <c:forEach items="${requestScope.customers}" var="c" varStatus="s">
        index:${s.index};isFirst:${s.first}; ${c.age } -- ${c.id } -- ${c.name }<br/>
    </c:forEach>
```

```jsp
    <c:forEach begin="0" end="4"  var="num">
        ${num}<br/>   <!--会输出 1 2 3 4 -->
    </c:forEach> 
```



- `<c:forTokens>`用来浏览**字符串**中所有的成员，其成员是由定义符号(delimiters)所分隔的。知道即可。属性：
  - `var`：存放当前指向的成员。不支持EL
  - `items`：被迭代的字符串
  - `delims`：定义用来分割字符串的字符，不支持EL。可以设置多个，间后面例子
  - `varStatus`：用于存放当前指向的成员信息的变量名。不支持EL。也有属性：
    - `index`：当前成员的索引；
    - `count`：指到的成员的总数
    - `first`：当前成员是否为第一个成员
    - `last`：当前成员是否为最后一个成员
  - `begin`：开始的位置，默认值为0。若`begin`>`end`会产生空的结果。
  - `end`：结束的位置，默认值为最后一个成员
  - `step`：每次迭代的间隔数，默认值为1。

```jsp
    <c:forTokens items="a,b,c,e,f,d" delims="," var="item">
        ${item}<br/>  <!--输出 a b c e f d-->
    </c:forTokens>
```

```jsp
    <c:forTokens items="bx,fS;ms5cm,zzk" delims=",;" var="item">
        ${item}<br/> <!--输出 bx fS ms5cm zzk-->
    </c:forTokens>
```



## URL操作

- `<c:import>`把其他静态或动态文件包含至当前JSP网页。了解即可
  - 与JSP行为`<jsp:include>`的区别：`<jsp:include>`只能包含和自己同一个Web应用下的文件，而`<c:import>`则可以包含任意位置的文件
  - 属性：
    - `url`：该地址的内容会被加到网页中。还有其他属性，由于不常用故没有列出。

```jsp
<c:import url="https://www.bilibili.com/"/>
```



- `<c:redirect>`是当前JSP页面重定向到指定页面。使用较少

```jsp
<c:redirect url="/cookie.jsp"/><!--这里的 / 代表当前Web应用的根目录-->
response.sendRedirect("/cookie.jsp"); <!--这里的 / 代表当前Web站点的根目录-->
```

若 `/ `需要服务器（Servlet容器）进行内部解析, 则代表的就是 WEB 应用的根目录；若是交给浏览器了, 则 `/` 代表的就是站点的根目录。



- `<c:url>`产生一个字符串类型的 URL 地址，而不是一个超链接。较为常用
  - 可以根据 Cookie 是否可用来智能进行 URL 重写, 
  - 可以对 GET 请求的参数进行编码
  - 可以把产生的 URL 存储在域对象的属性中
  - 可以使用`<c:param>`为 URL 添加参数. `<c:url>`会对参数进行自动的转码. 
  - 属性：
    - `value`：执行的URL，`/`代表的是当前 WEB 应用的根目录
    - `var`：存储被包含的文件内容（以`String`类型写入），有该属性时，网址会被存到该属性中，而不会直接输出网址。不支持EL
    - `scope`：`var`的JSP范围，默认值为`page`。不支持EL

```jsp
    <c:url value="/cookie.jsp" var="testurl" scope="page">
        <c:param name="name" value="bx"/>
    </c:url>
    url:${pageScope.testurl}  <!--这里是用于测试结果的-->
```

上面的例子中，当阻止了所有的Cookie后，在浏览器输入测试的网址，这里是`http://localhost:8080/jstl/jstl.jsp`，网页输出内容为：url:/cookie.jsp;jsessionid=16E42B2EBE388577F7E09CAAF37BCC3F?name=bx



# 7.2 I18N格式化标签库

- `<fmt:bundle>`标签用于绑定数据源的properties文件

  ```jsp
  <fmt:bundle basename="源文件名(且不能带后缀哦)" prefix=""> 
      语句，代码等 
  </fmt:bundle>
  ```

- `<fmt:message>`标签用于从指定的资源文件中把指定的键值取来

  - 必须和`<fmt:bundle>`搭配使用！！
  - 可以配合`<fmt:param>`标签来进行设定

  ```jsp
  <fmt:message key="" [var="varname"] [bundle=""] [scope="page|..."] />  
  如果用到var的话就不会在页面直接输出，而需要用到<c:out>标签来进行页面  的输出
  ```

- `<fmt:setBundle>`标签用于设置默认的数据来源

  ```jsp
  <fmt:setBundle basename="" [ var=""]  [scope="" ]  />
  ```

- `<fmt:formatNumber>`标签用于根据设定的区域将数据格式化输出

- `<fmt:formatDate>`标签用于格式化输出日期和时间;

- `<fmt:parseDate>`标签用于把字符串类型的日期和时间转换成日期型数据类型;

- `<fmt:setTimeZone>`标签用于设定默认的时区;

- `<fmt:timeZone>`标签用于设定在本签体内有效的时区
















