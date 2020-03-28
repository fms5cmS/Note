# 6.0 EL表达式

EL（Expression Language）表达式，简化JSP的写法。

```jsp
username:<input type="text" name="username"
   value="<%=request.getParameter("username")==null ? "" : request.getParameter("username")%>"/>
username:<input type="text" name="username" value="${param.username}"/>
```

上面的两种写法是等价的，而第二种就是使用了EL表达式。



# 6.1 简单特性

所有的 EL 都是在`${ }`中的。

如果使用 JSP Scriptlet 来获取用户性别：

```jsp
User user = (User)session.getAttribute("user");
String sex = user.getSex();
```

EL表达式：

```jsp
${sessionScope.user.sex}  <!--从 Session 的范围中取得用户的性别--%>
```

所以， EL 的语法比起传统的 JSP Scriptlet 更方便简洁。

- 运算符

  - EL 提供了`.`和`[]`两种运算符来存取数据。这两种运算符可以单独使用，也可以混合使用。

    ```jsp
    ${sessionScope.user.sex}    <!--等价于-->
    ${sessionScope.user["sex"]}
    ```

  - 如果要取得的属性名称中包含一些特殊字符，如`.`或`_`等并非字母或数字的符号，就一定要使用 `[]`

    ```jsp
    ${user["My-Name"] }
    ```

- EL变量

  - EL存取变量数据很简单，如：`${scope.username}`，取出某一范围中名为username的变量。

  - 如果没有指定哪个范围的变量，**默认查找顺序：Page、Request、Session、Application范围**，只要找到变量，就直接回传不再继续找，如果全部范围都没有找到变量，回传`null`。

  - 所以上例会先在Page范围找username变量。

    | 属性范围    | EL中的名称       |
    | ----------- | ---------------- |
    | Page        | pageScope        |
    | Request     | requestScope     |
    | Session     | sessionScope     |
    | Application | applicationScope |

  - 如果要指定取出哪个范围的变量，用法如：`${pageScope.username}`

- 自动类型转换，如：（这里仅列出了很少的内容）

  ```jsp
  <%--score的值为89--%>
  score:${param.score + 11}      输出为100
  score:<%=request.getParameter("score") + 11%>   输出为8911
  ```

- 算术运算符

  | 算术运算符   | 范例                     | 结果 |
  | ------------ | ------------------------ | ---- |
  | `+`          | `${12+5}`                | 17   |
  | `-`          | `${12-5}`                | 7    |
  | `*`          | `${12*5}`                | 60   |
  | `/ `或 `div` | `${12/5}`或`${12 div 5}` | 2    |
  | `%` 或 `mod` | `${12%5}`或`${12 mod 5}` | 2    |

- 关系运算符：必须放在`${}`里面

  | 关系运算符 | 范例                   | 结果  |
  | ---------- | ---------------------- | ----- |
  | `==`或`eq` | `${5==5}`或`${5 eq 5}` | true  |
  | `!=`或`ne` | `${5!=5}`或`${5 ne 5}` | false |
  | `<`或`lt`  | `${3<5}`或`${3 lt 5}`  | true  |
  | `>`或`gt`  | `${3>5}`或`${3 gt 5}`  | false |
  | `<=`或`le` | `${3<=5}`或`${3 le 5}` | true  |
  | `>=`或`ge` | `${3>=5}`或`${3 ge 5}` | false |

  - eg：`${param.score > 60 ? "及格" : "不及格"}`

- 逻辑运算符

  | 逻辑运算符  | 范例        | 结果       |
  | ----------- | ----------- | ---------- |
  | `&&`或`and` | `${A && B}` | true/false |
  | `||`或`or`  | `${A || B}` | true/false |
  | `!`或`not`  | `${!A}`     | true/false |

- 其他运算符

  - `empty`运算符：判断值是否为`null`或空的。eg：`${empty param.name}`
    - 作用于集合时，如果集合不存在或集合中没有元素，值均为true
  - 三目运算符：`${A ? B : C}`



# 6.2 EL隐含对象

EL隐含对象共有11个。

| 隐含对象         | 类型                         | 说明                                                         |
| ---------------- | ---------------------------- | ------------------------------------------------------------ |
| pageContext ★    | javax.servlet.ServletContext | 表示此JSP的 pageContext 代表页面上下文                       |
| pageScope        | java.util.Map                | 取得Page范围的属性值`page.getAttribute(String name)`         |
| requestScope ★   | java.util.Map                | 取得Request范围的属性值`request.getAttribute(String name)`   |
| sessionScope ★   | java.util.Map                | 取得Session范围的属性值`session.getAttribute(String name)`   |
| applicationScope | java.util.Map                | 取得Application范围的属性值`application.getAttribute(String name)` |
| param ★          | java.util.Map                | 等价`request.getParameter(String name)`。返回`String`        |
| paramValues      | java.util.Map                | 等价`ServletRequest.getParameterValues(String name)`。返回`String[]` |
| header           | java.util.Map                | 等价`ServletRequest.getHeader(String name)`。返回`String`    |
| headerValues     | java.util.Map                | 等价`ServletRequest.getHeader(String name)`。返回`String[]`  |
| cookie           | java.util.Map                | 等价`HttpServletRequest.getCookies()`                        |
| initParam        | java.util.Map                | 等价`ServletContext.getInitParameter(String name)`。返回`String`。获取web.xml中`<context-param>`标签定义的参数 |



# 6.3 EL自定义函数

EL 自定义函数：在 EL 表达式中调用的某个 Java 类的**静态**方法，这个静态方法需在 web 应用程序中进行**配置**才可以被 EL 表达式调用。

为了简化在 JSP 页面操作字符串，JSTL中提供了一套EL自定义函数，这些自定义函数包含了JSP页面制经常要用到的字符串操作。不常用。

导入EL自定义函数标签

```jsp
<%@ taglib prefix="fn" uri="http://java.sun.com/jsp/jstl/functions" %>
```

使用：

```jsp
${fn:length(param.name)}
```



开发步骤：

1. 编写 EL 自定义函数映射的Java 类中的静态方法：这个 Java 类必须带有`public`修饰符，方法必须是这个类的带有 `public`修饰符的静态方法；

   ```java
   public class MyELFunction {
       public static String concat(String str1,String str2){
           return str1+str2;
       }
   }
   ```

2. 编写标签库描述文件(tld 文件), 在 tld 文件中描述自定义函数；

   ```xml
       <short-name>myfn</short-name>
       <uri>http://myfunction.com</uri>
   
       <function>
           <name>concat</name>
           <function-class>myELfunctions.MyELFunction</function-class>
           <function-signature>java.lang.String concat(java.lang.String,java.lang.String)</function-signature>
       </function>
   ```

3. 在 JSP 页面中导入和使用自定义函数

   ```jsp
   <%@ taglib prefix="myfn" uri="http://myfunction.com" %>
   
   ${myfn:concat("我喜欢", "fS")}
   ```

