# 5.0自定义标签

利用自定义标签，可以软件开发人员和页面设计人员合理分工：页面设计人员可以把精力集中在使用标签(HTML,XML或者JSP)创建网站上,而软件开发人员则可以将精力集中在实现底层功能上面,如国际化等，从而提高了工程生产力。

自定义标签就是用户定义的一种自定义的jsp标记 。当一个含有自定义标签的jsp页面被jsp引擎编译成servlet时，tag标签被转化成了对一个称为 标签处理类 的对象的操作。于是，当jsp页面被jsp引擎转化为servlet后，实际上tag标签被转化为了对tag处理类的操作。 开发标签，核心就是编写标签处理类。

需要额外导入的jar包

```xml
        <dependency>
            <groupId>javax.servlet.jsp</groupId>
            <artifactId>jsp-api</artifactId>
            <version>2.2</version>
            <scope>provided</scope>
        </dependency>
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



# 5.1 开发&应用

1. 编写完成标签功能的Java类（标签处理器）；

   - 实现`javax.servlet.jsp.tagext.SimpleTag`接口
   - 通常继承`javax.servlet.jsp.tagext.SimpleTagSupport`类，该类为上面接口的实现类，可以直接调用其对应的 getter 方法得到对应的 API 。

2. 编写标签库描述（.tld为后缀的xml文件）文件，在 tld 文件中对自定义标签进行描述；

   - 在WEB-INF中创建 tld 文件（IDEA中新建：XML Configuration Files——>JSP Tag Library Descriptor）

     ```xml
         <tlib-version>1.0</tlib-version>
     
         <!--建议在JSP页面使用的标签前缀-->
         <short-name>zzk</short-name>
         <!--作为tld文件的id，用来唯一标识当前的tld文件，多个tld文件的uri不能重复-->
         <uri>http://www.fms5cms.com/mytag/core</uri>
     
         <tag>
             <!--标签名：在JSP页面上使用标签时的名字-->
             <name>hello</name>
             <!--标签处理类-->
             <tag-class>mytag.HelloSimpleTag</tag-class>
             <!--标签体类型-->
             <body-content>empty</body-content>
         </tag>
     ```

   - tld文件可以放置在web应用程序的WEB-INF目录及其子目录中，但不能放置在WEB-INF目录下的classes和lib子目录中。tld 文件也可以放置在WEB-INF\lib目录下的jar包的META-INF目录及其子目录中

3. 在 JSP 页面中导入和使用自定义标签。

   1. 导入标签库（描述文件）

      ```jsp
      <%@ taglib prefix="zzk" uri="http://www.fms5cms.com/mytag/core" %>
      <%--对应tld文件中的<short-name>和<uri>--%>
      ```

   2. 使用自定义标签

      ```jsp
      <zzk:hello/>
      ```



# 5.2 SimpleTag接口API

- `public void setJspContext(JspContext pc)`：把代表 JSP 页面的`PageContext`对象传递给标签处理器对象。
  - `PageContext` 可以获取 JSP 页面的其他 8 个隐含对象，所以凡是 JSP 页面可以做的标签处理器都可以完成。
- `public void setParent(JspTag parent)`：把父标签处理器对象传递给当前标签处理器对象。
- `public JspTag getParent()`：获得标签的父标签处理器对象。
- `public void setJspBody(JspFragment jspBody)`：把代表标签体的`JspFragment`对象传递给标签处理器对象。
- `public void doTag()`：完成所有的标签逻辑。该方法可以抛出`javax.servlet.jsp.JspException, java.io.IOException`异常，用于通知web容器不再执行JSP页面中位于结束标记后面的内容。

实现`SimpleTag`接口的标签处理器类的生命周期：

![](https://github.com/fms5cmS/MyNote/raw/master/images/Web-SimpleTag-life.png)

从图中可以看到，`setJspContext`先于`doTag`被 JSP 引擎所调用，可把代表 JSP 引擎的`pageContext`传给标签处理器。

```java
private PageContext pageContext;
	
@Override
public void setJspContext(JspContext pc) {
	System.out.println(pc instanceof PageContext);   //true
	this.pageContext = (PageContext) pc;
}
```



# 5.3 SimpleTagSuppert类

只需要覆盖`public void doTag()`即可。

```java
	@Override
    public void doTag() throws JspException, IOException {
        PageContext pageContext = (PageContext) getJspContext();//获取PageContext对象
        JspWriter out = pageContext.getOut();
    }
```



# 5.4 标签类型

- 标签类型：
  - 空标签：`<zzk:hello/>`
  - 带属性的空标签：`<max num1="3" num2="5">`
  - 带内容的标签：`<greeting> hello </greeting>`
  - 带内容和属性的标签：`<greeting name="Tom"> hello </greeting>`
- 标签体类型通过 tld 文件中的`<body-content>`来指定，取值有三种：
  - `empty`：没有标签体；
  - `scriptless`(常用)：标签体可以包含 EL 表达式和JSP动作元素，但不能包含JSP的脚本元素；
  - `tagdependent`：表示标签体交由标签本身去解析处理。此时，在标签体中的所有代码都会原封不动的交给标签处理器，而不是将执行结果传递给标签处理器



## 带属性的自定义标签

1. 在标签处理器中定义 setter 方法。建议把所有的属性类型都设置为`String`类型。

   ```java
   private String value;
   private String count;
   
   public void setValue(String value) {
   	this.value = value;
   }
   
   public void setCount(String count) {
   	this.count = count;
   }
   ```

2. 在 tld 描述文件中来描述属性（放在`<tag>`标签内）

   ```xml
       <tag>
           <name>hello</name>
           <tag-class>mytag.HelloSimpleTag</tag-class>
           <body-content>empty</body-content>
           <!-- 描述当前标签的属性 -->
           <attribute>
               <!-- 属性名, 需和标签处理器类的 setter 方法定义的属性相同 -->
               <name>value</name>
               <!-- 该属性是否被必须 -->
               <required>true</required>
               <!-- rtexprvalue: runtime expression value
                   当前属性是否可以接受运行时表达式的动态值。
   				true可以用${ }获取，false则不可以 -->
               <rtexprvalue>true</rtexprvalue>
           </attribute>
           <attribute>
               <name>count</name>
               <required>false</required>
               <rtexprvalue>false</rtexprvalue>
           </attribute>
       </tag>
   ```

3. 在页面中使用属性, 属性名同 tld 文件中定义的名字

   ```jsp
   <zzk:hello value="${param.name }" count="10"/>
   ```



## 带标签体的自定义标签

- `JspFragment `对象封装了标签体的内容；
- 若配置的标签含有标签体, 则 JSP 引擎会调用`setJspBody()`方法把`JspFragment`传递给标签处理器类，在 `SimpleTagSupport`中还定义了一个`getJspBody() `方法, 用于返回`JspFragment`对象；
- `JspFragment`的`invoke(Writer out)`方法: 把标签体内容从`Writer`中输出, 若为`out=null`, 则等同于`invoke(getJspContext().getOut())`, 即直接把标签体内容输出到页面上；
  - 有时, 可借助`StringWriter` 在标签处理器类中先得到标签体的内容（见下面的例子）。



1. 标签实现类

   ```java
   public class TestJspFragmentTag extends SimpleTagSupport {
       @Override
       public void doTag() throws JspException, IOException {
           JspFragment bodyContent = getJspBody();
           
           //1. 利用 StringWriter 得到标签体的内容.
           StringWriter sw = new StringWriter();
           bodyContent.invoke(sw);
   
           //2. 把标签体的内容都变为大写
           String content = sw.toString().toUpperCase();
   
           //3. 获取 JSP 页面的 out 隐含对象, 输出到页面上
           getJspContext().getOut().print(content);
       }
   }
   ```

2. tld文件

   ```xml
       <tlib-version>1.0</tlib-version>
           <short-name>fs</short-name>
           <uri>http://mycompany.com</uri>    
       <tag>
           <name>fragment</name>
           <tag-class>mytag.TestJspFragmentTag</tag-class>
           <body-content>scriptless</body-content>
       </tag>
   ```

3. 使用：需要导入标签！

   ```jsp
   <fs:fragment>abc</fs:fragment>
   ```



## 实现简单的`<c:forEach>`

使用案例：

```jsp
    <c:forEach items="${requestScope.customers}" var="customer">
        ${customer.id}~~~~${customer.name}~~~${customer.age} 
    </c:forEach>
```



- 两个属性: `items`(集合类型, `Collection`), `var`(`String`类型)；
- `doTag`: 
  - 遍历`items`对应的集合
  - **把正在遍历的对象放入到`pageContext`中**, 键: `var`, 值: 正在遍历的对象. 
  - 把标签体的内容直接输出到页面上

标签处理器：

```java
public class forEachTag extends SimpleTagSupport {
    private Collection<?> items;
    private String var;

    public void setItems(Collection<?> items) {
        this.items = items;
    }

    public void setVar(String var) {
        this.var = var;
    }

    @Override
    public void doTag() throws JspException, IOException {
        // 遍历 items 对应的集合
        if(items != null){
            for(Object obj: items){
                //把正在遍历的对象放入到 pageContext 中, 键: var, 值: 正在遍历的对象.
                getJspContext().setAttribute(var, obj);
                //把标签体的内容直接输出到页面上.
                getJspBody().invoke(null);
            }
        }
    }
}
```

描述文件：

```xml
    <short-name>fS</short-name>
    <uri>http://www.fS.cn</uri>

    <tag>
        <name>forEach</name>
        <tag-class>mytag.forEachTag</tag-class>
        <body-content>scriptless</body-content>
        <attribute>
            <name>items</name>
            <required>true</required>
            <rtexprvalue>true</rtexprvalue>
        </attribute>
        <attribute>
            <name>var</name>
            <required>true</required>
            <rtexprvalue>true</rtexprvalue>
        </attribute>
    </tag>
```

使用：需要导入标签！

```jsp
    <%
        List<Customer> customers = new ArrayList<Customer>();

        customers.add(new Customer(1, "AAA",17));
        customers.add(new Customer(2, "BBB",18));
        customers.add(new Customer(3, "CCC",19));
        customers.add(new Customer(4, "DDD",20));
        customers.add(new Customer(5, "EEE",21));

        request.setAttribute("customers", customers);
    %>
    <fS:forEach items="${requestScope.customers}" var="customer">
        --${customer.age } -- ${customer.id } -- ${customer.name } <br/>
    </fS:forEach>
```



## 带父标签的自定义标签

- 父标签无法获取子标签的引用, 父标签仅把子标签作为标签体来使用；
- 子标签可以通过`getParent()`方法来获取父标签的引用(需继承`SimpleTagSupport`或自实现`SimpleTag`接口的该方法)：若子标签的确有父标签, JSP 引擎会把代表父标签的引用通过`setParent(JspTag parent) `赋给标签处理器
  - 注意: 父标签的类型是`JspTag`类型. 该接口是一个空接口, 用于统一`SimpleTag`和`Tag`的. 实际使用需要进行类型的强制转换.
- 在 tld 文件中, 无需为父标签做额外配置，**子标签是以标签体的形式存在的**, 所以父标签的`<body-content></body-content>`需设置为`scriptless`

标签处理器：

```java
public class ParentTag extends SimpleTagSupport {
    private String name = "fS";
    public String getName() {   return name;    }

    @Override
    public void doTag() throws JspException, IOException {
        System.out.println("父标签处理器的name："+name);
        getJspBody().invoke(null);
    }
}
```

```java
public class SonTag extends SimpleTagSupport {
    @Override
    public void doTag() throws JspException, IOException {
        //1. 得到父标签的引用
        JspTag parent = getParent();

        //2. 获取父标签的 name 属性
        ParentTag parentTag = (ParentTag) parent;
        String name = parentTag.getName();

        //3. 把 name 值打印到 JSP 页面上.
        getJspContext().getOut().print("子标签输出name: " + name);
    }
}
```

描述文件：

```xml
    <short-name>fS</short-name>
    <uri>http://www.mksd.com</uri>
    
    <tag>
        <name>parent</name>
        <tag-class>mytag.ParentTag</tag-class>
        <body-content>scriptless</body-content>
    </tag>
    <tag>
        <name>son</name>
        <tag-class>mytag.SonTag</tag-class>
        <body-content>empty</body-content>
    </tag>
```

使用：需要导入标签！

```jsp
    <!-- 父标签打印name到控制台.  -->
    <fS:parent>
        <!-- 子标签以父标签的标签体存在,  子标签把父标签的name属性打印到 JSP 页面上.  -->
        <fS:son/>
    </fS:parent>
```



## 实现简单的`<c:choose>`

使用案例：

```jsp
<c:choose>
	<c:when test="${param.age > 24}">大学毕业</c:when>
	<c:when test="${param.age > 20}">高中毕业</c:when>
	<c:otherwise>高中以下...</c:otherwise>
</c:choose>
```

- 三个标签：`<choose>`,`<when>`,`<otherwise>`
  - `<when>`标签有一个`boolean`类型的属性: `test`
  - `<choose>`是`<when>`和`<otherwise>`的父标签
  - `<when>`在`<otherwise>`之前使用
  - 在`<choose>`中定义一个 "全局" 的`boolean`类型的 flag: 用于判断子标签在满足条件的情况下是否执行
    - 若`<when>`的`test==true`, 且`<when>`的父标签的`flag==true`, 则执行`<when>`的标签体(正常输出标签体的内容), 同时设置`flag=false`
    - 若`<when>`的`test==true`, 且`<when>`的父标签的`flag==false`, 此时`<when>`已经执行过一次故不再执行标签体. 
    - 若`flag==true`, `<otherwise>`执行标签体. 

标签处理器：

```java
public class Choose extends SimpleTagSupport {
    private boolean flag = true;

    public boolean isFlag() {   return flag;   }
    public void setFlag(boolean flag) {  this.flag = flag;  }

    @Override
    public void doTag() throws JspException, IOException {
        //由于子标签是父标签的标签体，所以这里必须写！
        getJspBody().invoke(null);
    }
}
```

```java
public class When extends SimpleTagSupport {
    private boolean test;

    public void setTest(boolean test) {   this.test = test;  }

    @Override
    public void doTag() throws JspException, IOException {
        Choose choose = (Choose)getParent();
        boolean flag = choose.isFlag();
        if(test==true && flag==true){
            getJspBody().invoke(null);
            choose.setFlag(false);
        }
    }
}
```

```java
public class Otherwise extends SimpleTagSupport {
    @Override
    public void doTag() throws JspException, IOException {
        Choose choose = (Choose)getParent();
        boolean flag = choose.isFlag();
        if(flag){
            getJspBody().invoke(null);
        }
    }
}
```

描述文件：

```xml
    <!--自定义choose标签-->
    <tag>
        <name>choose</name>
        <tag-class>mytag.Choose</tag-class>
        <body-content>scriptless</body-content>
    </tag>
    <tag>
        <name>when</name>
        <tag-class>mytag.When</tag-class>
        <body-content>scriptless</body-content>
        <attribute>
            <name>test</name>
            <required>true</required>
            <rtexprvalue>true</rtexprvalue>
        </attribute>
    </tag>
    <tag>
        <name>otherwise</name>
        <tag-class>mytag.Otherwise</tag-class>
        <body-content>scriptless</body-content>
    </tag>
```

使用：需要导入标签！可以以`http://localhost:8080/tag/testParent.jsp?age=24`这样的方式传入参数

```jsp
    <fS:choose>
        <fS:when test="${param.age > 24}">大学毕业</fS:when>
        <fS:when test="${param.age > 20}">高中毕业</fS:when>
        <fS:otherwise>高中以下...</fS:otherwise>
    </fS:choose>
```

