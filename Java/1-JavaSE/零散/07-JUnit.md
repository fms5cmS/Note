# 开始

Maven 导入：

```xml
<dependency>
    <groupId>junit</groupId>
    <artifactId>junit</artifactId>
    <version>4.12</version>
    <scope>test</scope>
</dependency>
```

**Keeps the bar green,to keeps the code clean.** 

测试结果如果是绿条就表示代码正确，如果是红条的话：

1. Failure是指测试失败；
2. Error是指测试程序本身出错。



测试相关的约定：

1. 类放在`test`包中；
2. 类名用`XXXTest`结尾；
3. 方法用`testMethod`命名。



# 断言

静态导入`Assert`包从而可以在不写`Assert`这一类名的情况下，调用`Assert`的静态方法：

```java
import static org.junit.Assert.*;
```

 如果参数中有`String message`，则表示这里测试失败后输出的错误信息为 `message`。 具体参数见API或源码。如：`AssertEquals()`方法；`assertTrue()`方法；`assertFalse()`方法。 

```java
assertTrue("z is too small",z > 9);    //前面的字符串作为测试失败的输出信息
```



**使用`hamcrest`断言**

`assertThat( actual ,  matcher )`可以替代上面的所有方法，其中`matcher`是 `org.hamcrest.Matcher` 的对象。示例：

```java
equalTo()可以比较两个对象，is()可以比较两个值
// allOf(大于1，小于15)就是说()里面的条件必须都满足
assertThat( n, allOf( greaterThan(1), lessThan(15) ) ); 
// anyOf(大于16，小于8)满足()里面的任意一个条件
assertThat( n, anyOf( greaterThan(16), lessThan(8) ) );  
assertThat( n, anything() );                            //anything()任意
assertThat( str, is( "bjsxt" ) );
assertThat( str, not( "bjxxt" ) );
assertThat( str, containsString( "bjsxt" ) );
assertThat( str, endsWith("bjsxt" ) );
assertThat( str, startsWith( "bjsxt" ) );
assertThat( n, equalTo( nExpected ) );
assertThat( str, equalToIgnoringCase( "bjsxt" ) );
assertThat( str, equalToIgnoringWhiteSpace( "bjsxt" ) );
assertThat( d, closeTo( 3.0, 0.3 ) );                    //closeTo()接近3.0，误差不超过0.3
assertThat( d, greaterThan(3.0) );
assertThat( d, lessThan (10.0) );
assertThat( d, greaterThanOrEqualTo (5.0) );
assertThat( d, lessThanOrEqualTo (16.0) );
assertThat( map, hasEntry( "bjsxt", "bjsxt" ) );
assertThat( iterable, hasItem ( "bjsxt" ) );
assertThat( map, hasKey ( "bjsxt" ) );
assertThat( map, hasValue ( "bjsxt" ) );
```



# 相关注解

- **@Test** 
  - 测试注解，指示该公共无效方法它所附着可以作为一个测试用例。
- **@Before** 
  - 该注解表明，该方法必须在类中的**每个测试之前执行**，以便执行测试某些必要的先决条件。（每次执行方法前都会执行）。
- **@BeforeClass**
  - 该注解指出这是**附着在静态方法**必须执行一次，并在类的所有测试之前(在类初始化之前就执行)。发生这种情况时一般是测试计算共享配置方法(如连接到数据库)。（最先执行）
  - 场景：测试时需要提前搭比较耗时间的环境，或需要取得很耗费时间的资源 如：建立数据库连接。
- **@After**
  - 该注解表明，该方法在执行**每项测试后**执行(如执行每一个测试后重置某些变量，删除临时变量等)（每次执行方法后都会执行）。
- **@AfterClass** 
  - 当需要执行所有的测试在JUnit测试用例类后执行，AfterClass注解可以使用以清理建立方法，(从数据库如断开连接)。注意：(类似于BeforeClass)的**方法必须定义为静态**。（最后执行）
  - 场景：测试时需要卸载环境，或需要释放资源 如：关闭数据库连接 
- **@Ignore**
  - 当想暂时禁用特定的测试执行可以使用忽略注释。每个被注解为@Ignore的方法将不被执行。
  - 场景：某个方法的模块还没有完成，先不测试这个方法。

