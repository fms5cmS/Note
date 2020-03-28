# 3.0 JSP指令

JSP指令（directive）告诉引擎如何处理JSP页面中的相关部分，用来设置与整个JSP页面相关的属性。 主要定义了page、include、taglib三种指令。 

语法：

```jsp
<%@ 指令 属性名="值 " %>            <!-- 属性名部分是大小写敏感的 -->
```

```jsp
<%@ page language="java" contentType="text/html; charset=UTF-8" pageEncoding="UTF-8"%>
```



# 3.1 page指令

page指令用于定义页面的依赖属性，比如脚本语言、error页面、缓存需求等等 ，无论page指令出现在JSP页面的任何地方，它==作用的都是整个JSP页面==。page指令==最好放在整个JSP页面的起始位置==。 

- 完整语法（下面仅列举了常用的属性）

  - 导入包

    ```jsp
    <%@ page import="java.text.DateFormat" %> 
    <!--指定当前JSP页面对应的Servlet需要导入的类-->
    ```

  - 指定当前页面的 session 隐含变量是否可用

    ```jsp
    <%@ page session="true / false"  %>     
    <!--访问当前页面时是否一定要生成 HttpSession 对象-->
    ```

  - error 相关的属性

    ```jsp
    <!--errorPage 指定若当前页面出现错误时的响应页面。其中“/”表示当前Web应用的根目录。-->
    <!--在响应error.jsp时，JSP引擎使用的是请求转发的方式。-->
    <%@ page errorPage="/error.jsp" %> 
    
    <!--isErrorPage 指定当前页面是否为错误处理页面，可以说明当前页面是否可以使用exception隐藏对象。需要注意的是：若指定isErrorPage=“true”，并使用exception的方法了，一般不建议能够直接访问该页面。-->
    <%@ page isErrorPage="false"  %> 
    ```

    还可以在web.xml文件中配置错误页面：

    ```xml
    <error-page>
        <!--指定出错的代码-->
        <error-code>404</error-code>
        <!--指定响应页面的位置-->
        <location>/hello.jsp</location>
    </error-page>
    
    <error-page>
        <!--指定异常的类型-->
        <exception-type>java.lang.ArithmeticException</exception-type>
        <location>/WEB-INF/erros.jsp</location>
    </error-page>
    ```

  - 指定 JSP 页面的响应类型

    ```jsp
    <%@ page  contentType="text/html; charset=ISO-8859-1"  %>
    <!--指定当前JSP页面的响应类型。实际调用的是
    response.setContentType("text/html; charset=ISO-8859-1")
    通常情况下，对于JSP页面而言其取值均为"text/html; charset=ISO-8859-1"。
    charset指定返回页面（响应页面）的指定字符编码，通常取值为UTF-8
    可以在tomcat/conf下的web.xml文件中查询每种文件类型的写法-->
    ```

  - 指定当前 JSP 页面的字符编码

    ```jsp
    <!--通常情况下该值和contentType中的charset一致-->
    <%@ page   pageEncoding="ISO-8859-1" %>
    ```

  - 指定当前 JSP 页面是否可用 EL 表达式

    ```jsp
    <%@ page   isELIgnored="false" %>    <!--通常取值为true。-->
    ```

| **属性**           | **描述**                                            |
| ------------------ | --------------------------------------------------- |
| buffer             | 指定out对象使用缓冲区的大小                         |
| autoFlush          | 控制out对象的 缓存区                                |
| contentType        | 指定当前JSP页面的MIME类型和字符编码                 |
| errorPage          | 指定当JSP页面发生异常时需要转向的错误处理页面       |
| isErrorPage        | 指定当前页面是否可以作为另一个JSP页面的错误处理页面 |
| extends            | 指定servlet从哪一个类继承                           |
| import             | 导入要使用的Java类                                  |
| info               | 定义JSP页面的描述信息                               |
| isThreadSafe       | 指定对JSP页面的访问是否为线程安全                   |
| language           | 定义JSP页面所用的脚本语言，默认是Java               |
| session            | 指定JSP页面是否使用session                          |
| isELIgnored        | 指定是否执行EL表达式                                |
| isScriptingEnabled | 确定脚本元素能否被使用                              |



# 3.2 include指令

include指令用于通知 JSP 引擎在翻译当前 JSP 页面时将其他文件中的内容合并进当前 JSP 页面转换成的Servlet 源文件中，这种在源文件级别进行引入的方式称为==静态引入，当前 JSP 页面与静态引入的页面紧密结合为一个 Servlet。==

 语法：

```jsp
<%@ include file="relativeURL" %>    <!--其中file属性用于指定被引入文件的相对路径-->
```

```jsp
<%@ include file="b.jsp" %>      <!-- 在a.jsp中包含b.jsp -->
```

注意：如果以 `/ `开头，表示相对于当前Web应用的根目录（不是站点根目录），否则的话，相对于当前文件。推荐使用` / `的方式来包含页面。使用绝对路径肯定没问题，使用相对路径可能出问题。



# 3.3 taglib指令

taglib 指令用于引入一个自定义标签集合的定义，包括库路径、自定义标签。语法：

```jsp
<%@ taglib uri="uri" prefix="prefixOfTag" %>
<!--uri属性确定标签库的位置，prefix属性指定标签库的前缀。-->
```

