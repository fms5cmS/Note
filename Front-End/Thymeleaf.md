常见 Java 模板引擎：JSP、Velocity、Freemarker、Thymeleaf。能够处理 HTML、XML、JavaScript、CSS 甚至纯文本。使用 OGNL(Object-Graph Navigation Language)，在Spring应用中可以使用 SpEL。

SpringBoot 推荐使用 Thymeleaf：语法更简单，功能更强大。



# 语法

## 标签

`th:...`标签：

==`th:任意html属性`== 替换原生属性的值。如：`th:text`改变当前元素里面的文本内容

```html
<div id="div01" th:id="${hello}">这是欢迎信息</div>
<!--运行程序后，查看网页源代码发现原来的id(div01)也被替换了-->
```

其他标签(以下标签的**优先级从上往下逐渐降低**，即多个`th:*`放在同一个HTML标签中的执行顺序)：

| 特性                                                         | 属性                                                     |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| Fragment inclusion（片段包含，类似 jsp:include）             | th:insert<br/>th:replace                                 |
| Fragment iteration（遍历，类似 c:foreach）                   | th:each                                                  |
| Conditional evaluation（条件判断， c:if）                    | th:if <br/>th:unless <br/>th:switch<br/>th:case          |
| Local variable definition（声明变量，c:set）                 | th:object<br/>th:with                                    |
| General attribute modification<br/>（任意属性修改，支持前面添加-prepend、后面添加-append） | th:attr <br/>th:attrprepend<br/>th:attrappend            |
| Specific attribute modification（修改指定属性默认值）        | th:value<br/>th:href<br/>th:src ...                      |
| Text (tag body modification) （修改标签体内容）              | th:text（会转义特殊字符）<br/>th:utext（不转义特殊字符） |
| Fragment specification（声明片段）                           | th:fragment                                              |
| Fragment removal                                             | th:remove                                                |



## 表达式

### 变量表达式

语法：`${}`

- 获取对象的属性、调用方法

- 通过`#`来使用内置的基本对象：

  ```
  #ctx : 上下文对象
  #vars: the context variables.
  #locale : 直接访问与java.util.Locale关联的当前的请求
  #request : (only in Web Contexts) the HttpServletRequest object.
  #response : (only in Web Contexts) the HttpServletResponse object.
  #servletContext : (only in Web Contexts) the ServletContext object.
  ```

- 内置的一些工具对象：

  ```
  #execInfo : information about the template being processed.
  #messages : methods for obtaining externalized messages inside variables expressions, in the same way as theywould be obtained using #{…} syntax.
  #uris : methods for escaping parts of URLs/URIs Page 20 of 106
  #conversions : methods for executing the configured conversion service (if any).
  #dates : methods for java.util.Date objects: formatting, component extraction, etc.
  #calendars : analogous to #dates , but for java.util.Calendar objects.
  #numbers : methods for formatting numeric objects.
  #strings : methods for String objects: contains, startsWith, prepending/appending, etc.
  #objects : methods for objects in general.
  #bools : methods for boolean evaluation.
  #arrays : methods for arrays.
  #lists : methods for lists.
  #sets : methods for sets.
  #maps : methods for maps.
  #aggregates : methods for creating aggregates on arrays or collections.
  #ids : methods for dealing with id attributes that might be repeated (for example, as a result of an iteration).
  ```



### 消息表达式

语法：`#{}`

也称为文本外部化、国际化(i18n)



### 选择表达式

语法：`*{}`。

和变量表达式在功能上一致，区别在于选择表达式是在**当前选择的对象**而不是整个上下文变量映射上执行。

必须配合`th:object`使用！！

```html
<div th:object="${session.user}">   <!--暂存user对象，后面使用*{}来直接使用该对象-->
    <!--*{} 取的是上面暂存的user对象内的属性，而不是在整个上下文取-->
    <p>Name: <span th:text="*{firstName}">Sebastian</span>.</p>
    <p>Surname: <span th:text="*{lastName}">Pepper</span>.</p>
    <p>Nationality: <span th:text="*{nationality}">Saturn</span>.</p>
</div>
```



### 链接表达式

语法：`@{}`

通常用于定义URL。

链接表达式可以是相对的，此时，应用程序上下文将不会作为 URL 的前缀：

```html
<a th:href="@{../documents/report}"> </a>
```

也可以是服务器相对，同样没有应用程序上下文前缀：

```html
<a th:ref="@{~/contents/main}"> </a>
```

和协议相对（就像绝对路径，但浏览器将使用显示的页面中使用的相同的HTTP或HTTPS协议）：

```html
<a th:ref="@{//static.mycompany.com/res/initial}"> </a>
```

链接表达式也可以是绝对的：

```html
<a th:ref="@{http://www.mycompany.com/main}"></a>
```



### 分段表达式

语法：`~{}`

需要配合`th:insert`或`th:replace`标签使用！

```html
<div th:insert="~{commons :: main}">...</div>
```



## 字面量

文本：使用`''`来包含，如：`'one text'`

数字：直接使用数字即可，也可进行运算

布尔值：`true`、`false`

Null：`null`



## 其他操作

### 运算

算术运算：

```properties
Arithmetic operations（数学运算符）:
    Binary operators: + , - , * , / , %
    Minus sign (unary operator): -
```

布尔运算：

```properties
Boolean operations（布尔运算）:
    Binary operators: and , or
    Boolean negation (unary operator): ! , not
```

条件运算：

```properties
Conditional operators（条件运算）:
    If-then: (if) ? (then)
    If-then-else: (if) ? (then) : (else)
    Default: (value) ?: (defaultvalue)
```

比较运算：

```properties
Comparisons and equality（比较运算）:
    Comparators: > , < , >= , <= ( gt , lt , ge , le )
    Equality operators: == , != ( eq , ne )
```



### 文本操作

```properties
Text operations（文本操作）:
    String concatenation: +
    Literal substitutions: |The name is ${name}|
```



### 无操作

```properties
Special tokens（特殊操作）:
	No-Operation: _  （什么也不做）
```



## 注释

- 标准 HTML/XML 注释

- Thymeleaf 解析器级注释块：`<!--/*-->`和`<!--*/-->`包含的内容会在解析时被删除

  - 静态打开时该注释不生效

- 原型注释块：

  - 当模板静态打开时(如原型设计)，该注释块注释的代码被注释，而在模板执行时，这些被注释的代码就能被显示出来

    ```html
    <span>hello!</span>
    <!--/*/
    	<div th:text="${...}">
    	...
    	</div>
    /*/-->
    <span>bye!</span>
    ```

    

## 内联

内联表达式：

- `[[...]]`对应于`th:text`，会对特殊字符转义

  ```html
  <p>The message is "[[${msg}]]"</p>
  ```

  会变成：

  ```html
  <p>The message is "This is &lt;b&gt; great! &lt;/b&gt;"</p>
  ```

- `[(...)]`对应于`th:utext`，不对特殊字符转义

  ```html
  <p>The message is "[(${msg})]"</p>
  ```

  会变成：

  ```html
  <p>The message is "This is <b>great!</b>"</p>
  ```



禁用内联，如就是想输出`[[...]]`或`[(...)]`：在标签中加入`th:inline="none"`。

JavaScript、CSS 也可以使用内联







## 案例

```html
<!--有转义-->
<div th:text="${hello}"></div>
<!--没有转义-->
<div th:utext="${hello}"></div>
<hr/>
<!--th:each每次遍历时都会生成当前所在标签。users有3个值，所以这里是3个h4标签-->
<h4 th:text="${user}" th:each="user:${users}"></h4>
<hr/>
<h4>
    <!--[[...]]是thymeleaf的行内写法。这里有3个span标签-->
    <span th:each="user:${users}">[[${user}]]</span>
</h4>
```



# 公共元素抽取

如果两个页面在切换时有共同的内容，就可以对这些前后没有变化的公共元素进行抽取。步骤：

1. 抽取公共片段

   ```html
   <div th:fragment="copy">  这里的copy是随意的，第二步会使用
   	&copy; 2011 The Good Thymes Virtual Grocery
   </div>
   ```

2. 引入公共片段

   ```html
   <div th:insert="~{footer :: copy}"></div>   这里footer是上一步代码所在文件名
   ```

   - 这里有两种写法：

     ```
     ~{templatename::selector}     模板名::选择器(这里的选择器与CSS中相同的使用)
     ~{templatename::fragmentname}    模板名::片段名
     ```

   - 模板名会使用Thymeleaf的前后缀配置规则进行解析

3. 默认效果:

   - `insert`的公共片段在div标签中
   - 如果使用`th:insert`等属性进行引入，可以不用写`~{}`：
   - 行内写法可以加上：`[[~{}]]`或`[(~{})]`；



三种引入公共片段的 th 属性：

- `th:insert`：将公共片段整个插入到声明引入的元素中
- `th:replace`：将声明引入的元素替换为公共片段
- `th:include`：将被引入的片段的内容包含进这个标签中，3.x 后不再使用

```html
<footer th:fragment="copy">
	&copy; 2011 The Good Thymes Virtual Grocery
</footer>
```

使用三种方式及其效果：

```html
<div th:insert="footer :: copy"></div> 效果如下：
<div>
    <footer>
    &copy; 2011 The Good Thymes Virtual Grocery
    </footer>
</div>
```

```html
<div th:replace="footer :: copy"></div>  效果如下：
<footer>
	&copy; 2011 The Good Thymes Virtual Grocery
</footer>
```

```html
<div th:include="footer :: copy"></div>  效果如下：
<div>
	&copy; 2011 The Good Thymes Virtual Grocery
</div>
```



# SpringBoot整合Thymeleaf

SpringBoot引入Thymeleaf：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-thymeleaf</artifactId>
</dependency>

可以使用以下方法更换默认的版本
<properties>
    <thymeleaf.version></thymeleaf.version>
    <!--布局功能的支持程序-->
    <thymeleaf-layout-dialect.version></thymeleaf-layout-dialect.version>
</properties>
```

修改application.properties：

```properties
spring.thymeleaf.encoding=UTF-8
# 热部署静态文件
spring.thymeleaf.cache=false
# 使用HTML5标准
spring.thy,eleaf.mode=HTML5
```





查看`ThymeleafProperties`中默认的配置规则：

```java
@ConfigurationProperties(prefix = "spring.thymeleaf")
public class ThymeleafProperties {
	private static final Charset DEFAULT_ENCODING = StandardCharsets.UTF_8;

    //只要把 HTML 页面放在 classpath:/templates/ 下，thymeleaf就可以自动渲染
	public static final String DEFAULT_PREFIX = "classpath:/templates/";
	public static final String DEFAULT_SUFFIX = ".html";
	...
}
```

使用：

1. 导入 thymeleaf 的名称空间，在编写thymeleaf时会有语法提示

   ```html
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
   ```

2. 使用 thymeleaf 的语法

   ```html
   <!DOCTYPE html>
   <html lang="en" xmlns:th="http://www.thymeleaf.org">
   <head>
       <meta charset="UTF-8">
       <title>Title</title>
   </head>
   <body>
       <h1>SUCCESS!</h1>
       <!--th:text 将div里面的文本内容替换为指定的内容。-->
       <div th:text="${hello}">这是欢迎信息</div>
   </body>
   </html>
   ```

