

Spring提供了三种bean的装配机制：

- 显式装配：
  - XML配置文件装配；
  - Java配置类装配；
- 隐式装配：通过组件扫描+自动装配完成。

优先选择建议：隐式装配 > Java配置类装配 > XML配置文件装配



# 1.1 隐式装配

## 基础

### 组件扫描

组件扫描（component scanning）：Spring能对classpath下指定的包自动扫描，侦测和实例化具有特定注解的组件，然后在IoC容器中自动为其创建一个 bean。

- 特定注解包括：
  - `@Component`：
  - `@Repository`：标识持久层组件
  - `@Service`：标识服务层（业务层）组件
  - `@Controller`：标识表现层组件
- 对于扫描到的组件，Spring有**默认的命名策略**：使用非限定类名，第一个字母小写，也可以在上述注解中通过`value`属性值来标识组件的名称（相当于给定了在获取当前 bean 实例时的 id ）。
- 组件扫描默认是禁用的，需要在配置类中使用`@ComponentScan`注解开启组件扫描。部分属性：
  - **如果不设置该注解的任何属性，则默认扫描当前配置类所在的包**；
  - `String[] value`：指定扫描的包。该属性与`basePackages`是互为别名的两个属性。扫描多个包时，包之间使用`,`分隔，该方法是类型不安全的；
  - 推荐使用`basePackageClasses`属性来指定扫描的包，Spring会扫描该类所在的包，多个类使用`,`分隔。可以在包中创建一个专门用于扫描的空标记接口(marker interface)。

注解方式：

```java
@Configuration	
@ComponentScan(basePackageClasses = ScanSoundsystem.class)
public class CDPlayerConfig {    }
```

XML配置文件方式：

```xml
<!--Spring 容器将会扫描base-package包里及其子包中的所有类；-->
<!-- 通过 resource-pattern 指定扫描的资源 这里扫描repository下的所有资源-->
<context:component-scan base-package="annotation"
          resource-pattern="repository/*.class"></context:component-scan>
```



### 自动装配

自动装配（autowiring）：Spring利用 DI 完成对 IoC 容器中各个组件依赖关系的赋值。

- 将`@Autowired`标注在需要装配的属性，IoC容器会为其自动装配。



## 进阶

### 组件扫描

- `@ComponentScan`注解的其他属性：

  - `Filter[] excludeFilters`：指定扫描时按照规则排除的组件

    - `@ComponentScan.Filter` 过滤: 
      - `type`：过滤规则（例子中是按注解排除） ;
      - `classes`：排除的注解 

  - `Filter[] includeFilters`：指定扫描时只包含哪些组件，必须配合 `useDefaultFilters = false`来使用。

  - `includeFilters`和`excludeFilters`的过滤方式：

    | 类别                           | 说明                                                         |
    | ------------------------------ | ------------------------------------------------------------ |
    | ` FilterType.ANNOTATION `      | 按照是否标注了某个注解进行过滤                               |
    | ` FilterType.ASSIGNABLE_TYPE ` | 按照给定类型过滤                                             |
    | ` FilterType.ASPECTJ `         | 按照 Aspectj 表达式进行过滤                                  |
    | ` FilterType.REGEX `           | 采用正则表达式根据类名过滤                                   |
    | ` FilterType.CUSTOM`           | 按照自定义规则过滤，该类必须实现`org.springframework.core.type.TypeFilter`接口 |

```java
@Configuration
@ComponentScan(value = "com.zzk",excludeFilters = {
    @ComponentScan.Filter(type = FilterType.ANNOTATION,classes ={Controller.class,Service.class})})
public class MainConfig { /**配置内容*/ }
```

```java
@Configuration
@ComponentScan(value = "com.zzk",useDefaultFilters = false,includeFilters = {
    @ComponentScan.Filter(type = FilterType.ASSIGNABLE_TYPE,classes ={BookService.class})})
public class MainConfig { /**配置内容*/ }
```

```java
/** 自定义过滤规则*/
public class MyFilter implements TypeFilter {
   /**
    * @param metadataReader ：读取到当前正在扫描的类的信息
    * @param metadataReaderFactory ：可以获取到其他任何类的信息
    */
  @Override
  public boolean match(MetadataReader metadataReader, MetadataReaderFactory
metadataReaderFactory) throws IOException {
    String message = "er";
    //获取当前类注解的信息
    AnnotationMetadata annotationMetadata =
metadataReader.getAnnotationMetadata();
    //获取当前正在扫描的类的信息
    ClassMetadata classMetadata = metadataReader.getClassMetadata();
    //获取当前类的资源（如：路径）
    Resource resource = metadataReader.getResource();
    String className = classMetadata.getClassName();
    System.out.println("--->" + className);
    //所有类名包含了 “er” 的都可以通过扫描
    if(className.contains(message)){
      return true;
   }
    return false;
 }
```



### 自动装配

- `@Autowired`注解：Spring定义的注解

  - 优先按照类型在容器中找对应的组件，找到就赋值；
    - 自动装配默认一定要将属性赋值成功，没有就会报错。如果某一属性允许不被赋值，可以使用`@Autowiredd`的`required`属性来设置，此时找不到可以装配的组件的话不报错。
    - 注意，当设置`required=false`后，如果代码中没有`null`检查的话，这个未装配的属性可能会在使用时抛出`NullPointerException`。
  - 如果找到多个相同类型的组件，
    - 将属性的名称（bookDao）作为组件的id去容器中找。
    - 可使用`@Qualifier`注解明确指定要装配的组件的 id 。
    - 可使用`@Primary`注解指定当前标注的Bean在进行自动装配时为首选注入的 Bean。

  ```java
  @Service
  public class BookService {
      @Qualifier("bookDao")  //若有多个BookDao类，则指定要装配其中id=bookDao的组件
      @Autowired(required = false) 
      private BookDao bookDao2;
  }
  ```

  ```java
  @Configuration
  public class MainConfigOfAutowired {
      @Primary   //如果有多个BookDao的话，该bean为首选
      @Bean
      public BookDao bookDao(){
          return new BookDao();
      }
  }
  ```

  - `@Autowired`可以标注在构造器、参数、方法、属性上
    - 标注在方法上，IoC容器创建方法所在类的对象时，方法使用的参数会从IoC容器中获取
  - `@Autowired`也可用在数组类型的属性上，Spring将会把所有匹配的Bean进行自动装配
  - `@Autowired`注解也可用在集合属性上，Spring读取该集合的类型信息，然后自动装配所有兼容的Bean
  - `@Autowired`用在Map上时，若该`Map`的键值为`String`，Spring将自动装配与之`Map`值类型兼容的Bean，用Bean名称作为键值 

- `@Resource`注解（JSR-250）和`@Inject`注解（JSR-330）

  - 这两个注解是java规范的注解。
  - `@Resource`注解要求提供一个Bean名称的属性，若该属性为空，自动采用标注处的变量或方法名作为Bean的名称；不支持`@Primary`、`required`功能
  - `@Inject`注解，和`@Autowired`注解一样也是按类型匹配注入的Bean，但没有`required`属性。需要导入`javax.inject`依赖。
  - 建议使用`@Autowired`注解 



# 1.2 Java配置类装配

Spring自动化配置很方便，但当我们需要将第三方库中的组件装配进项目中时，就必须使用显式装配了。而Java配置类相比于XML配置文件要更为强大、类型安全，且对重构友好。



## 基础

- 创建普通类，使用`@Configuration`注解表示当前类是一个配置类。如果要在配置类中声明一个bean：

  1. 编写一个方法，该方法会创建所需类型的实例并返回，
  2. 给方法添加`@Bean`注解。这个方法返回的对象即为要注册的bean。
     - 使用`@Bean`的`name`属性或`value`属性可为bean指定id
     - 如果不指定，该bean默认的 id 为方法名。

  - 如果bean A依赖于另一个bean B的话，装配A最简单的方式就是引用创建B的方法，具体见后面的示例。



```java
@Configuration
public class CDPlayerConfig {
	/**没有依赖的bean，创建一个返回该类型的方法，并标注@Bean注解，这里修改了默认id*/
    @Bean("infinte")
    public CompactDisc infiniteS(){
        return new InfiniteS();
    }
	//Spring对配置类会特殊处理，只要方法是给容器中添加组件的，该方法多次调用都只是在容器中找组件
    @Bean
    public CDPlayer cdPlayer(){
        return new CDPlayer(infiniteS());
    }
}
```

**注意：在上面的例子中，CDPlayer是通过引用了infiniteS()方法来为自身的属性注入值的，由于infiniteS()方法上标注了@Bean注解，Spring会拦截所有对这个方法的调用，确保所有调用该方法后返回的是同一个bean，而不是每次调用都创建一次bean！！！！！！！！！**



## 注入属性

使用`@Value`注解来给bean的属性赋值。在bean的属性上使用`@Value`注解来赋值，里面可以写：

1. 基本数值；
2. SpEL（见2.4）：`#{}` 
3. 属性占位符（见2.4）：`${}`：从环境变量、配置文件中获取值（引用外部配置文件见1.4）

```java
public class Person {
    @Bean
    public Person person(){
        @Value("bx")
        private String name;
        
        @Value("#{20-2}")
        private int age;
        
        //这里使引用了外部配置文件 person.properties 中的 person.nickName 值
        @Value("${person.nickName}")
        private String nickName;
        //。。。get、set方法、构造器、toString
        return new Person(name)
    }
}
```



自定义组件想要使用Spring容器底层的一些组件（`ApplicationContext`、`BeanFactory`等），只要自定义组件实现`Aware`接口对应的子接口即可，在创建对象时，会调用接口定义的方法注入相关的值。

`xxxAware`的功能是使用`xxxProcessor`实现的。

```java
public class Red implements ApplicationContextAware,BeanNameAware,EmbeddedValueResolverAware {
    /**保存下来便于使用*/
    private ApplicationContext applicationContext;

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
	
    @Override
    public void setBeanName(String name) {
        System.out.println("当前bean的id："+name);
    }
    /**解析字符串*/
    @Override
    public void setEmbeddedValueResolver(StringValueResolver resolver) {
        String s = resolver.resolveStringValue("当前系统为：${os.name}");
        System.out.println(s);
    }
}
```



# 1.3 XML配置文件装配

建议这部分仅用于维护已有的XML配置。

Spring的XML配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?> 
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
       http://www.springframework.org/schema/beans/spring-beans.xsd">

    <!--所有的bean都在这里配置-->
</beans>
```

在`<beans>`根元素内，使用`<bean>`元素来声明一个bean，其中`id`属性为该bean的id，`class`属性为该bean的全限定类名。当Spring发现这个`<bean>`元素时，会调用其默认构造器创建bean。

普通的bean使用`<bean>`元素来声明；而独立的集合bean需要导入 util-命名空间后才能声明。

- 外部集合bean，作为一个单独的bean，可在其他bean之间共享。

  - 需向`<beans>`元素中声明 util-命名空间

  - 示例：

    ```xml
         <util:list id="cars">
             <ref bean="car"/>
             <ref bean="car2"/>
         </util:list>
    ```



## 属性赋值的方式

当设值注入和构造器注入同时存在时，优先执行设值注入！！随着 Spring 容器的创建，容器会验证每个 Bean 的配置，但是**只有 Bean 被真正创建时，Bean 的属性才会被设置**。



### 设值注入

- 设值注入实质是通过bean的`setXxx()`方法来为其注入属性值或依赖的对象：
  - 每个`<property>`元素代表了bean的一个属性，`<property>`元素的属性：
  - `name`：指定类中的属性名；
  - `value`：为bean的属性注入字面量；
  - `ref`：为bean的属性注入一个引用的bean（容器中存在的bean），取值为引用的bean的 id 值。

```xml
<bean id="infinite" class="chapter2.soundsystem.InfiniteS">
    <property name="artist" ref="fS"/> 
    <property name="title" value="infinite synthesis"/>
</bean>
```



为了简化设值注入的编写，Spring 2.5 引入了 p-命名空间，在`<beans>`元素中声明该命名空间后，使用p-命名空间后的写法：`p:属性名="字面量值"`或`p:属性名-ref="引用bean的id"`

```xml
<bean id="infinite" class="chapter2.soundsystem.InfiniteS" p:title="infinite synthesis" p:artist="fS"/>
```

注：p-命名空间虽然写法更加简练，但相比于`<property>`元素而言，无法装配内部集合bean（即仅使用一次的集合bean）！！



### 构造器注入

- 通过构造方法注入bean的属性值或依赖的对象，保证了**bean实例在实例化后就可使用**。
  - 每个`<constructor-arg>`元素代表构造器的一个参数，`<constructor-arg>`元素的属性：
    - `ref`：为构造器的参数注入一个引用的bean（容器中必须存在），取值为引用的bean的 id 值。

```xml
<bean id="cdplayer" class="chapter2.soundsystem.CDPlayer">
    <constructor-arg ref="infinite"/>
</bean>
```



为了简化构造器注入的编写，Spring 3.0 引入了 c-命名空间，在`<beans>`元素中声明该命名空间后，使用c-命名空间后的写法：`c:属性名="字面量值"`或`c:属性名-ref="引用bean的id"`

```xml
<bean id="cdplayer" class="chapter2.soundsystem.CDPlayer" c:cd-ref="infinite"/>
```

注：与p-命名空间类似，c-命名空间也无法装配内部集合bean！！



**补充**

如果构造器有多个参数时：可以使用位置 index 和类型 type 以区分重载的构造器！注意：index 的数值从 0 开始，也可以使用 name 属性来直接给属性名对应的属性赋值

```xml
<bean id="car" class="spring.Car">
    <constructor-arg value="Audi" index="0"/>
    <constructor-arg name="address" value="ShangHai"/>
    <constructor-arg value="300000" type="double"/>
</bean>
```



### 特殊说明

- 注意：

  - 构造器注入可能会产生无解的循环依赖场景。如：A 类通过构造器注入来配置一个 B 类依赖，B 类也用构造器注入配置了一个 A 类的依赖，在运行时 IoC 容器检测到循环依赖，从而抛出`BeanCurrentlyInCreationException `异常。
  - 一种解决方法是：编辑那些通过 setter 而不是构造器配置的类的源代码；或者避免使用构造器注入而仅使用设值注入。

- 补充：

  - `<constructor-arg>`和`<property>`标签内可以嵌套使用`<idref>`标签，该标签与`<ref>`标签不同，后者是用来引用 bean 实例的，前者仅将 Bean 的 `id` 值赋给变量，运行效果上等同于`value`属性的作用。官方例子：

    ```xml
    <bean id="theTargetBean" class="..."/>
    
    <bean id="theClientBean" class="...">
        <property name="targetName">
            <!--这种用法可以使容器在部署阶段就能验证 theTargetBean 是否实际存在-->
            <idref bean="theTargetBean"/>
        </property>
    </bean>
    <!--等价于：-->
    <bean id="client" class="...">
        <!--只有在 client 被实例化时才能发现；如果这个 client 是一个prototype的bean，那么可能会在容器部署很长时间以后才能发现错误-->
        <property name="targetName" value="theTargetBean"/>
    </bean>
    ```



## 注入不同类型

下面的例子并没有列出Java类的实际细节，仅用于理解。



### 字面量的注入

字面值：用字符串表示的值，可以通过`<value>`元素标签或value属性进行注入基本数据类型及其包装类、String等类型都可以采取字面值注入的方式若字面值中包含特殊字符，可以使用==`<![CDATA[]]>`==把字面值包裹起来 :

```xml
<bean id="car2" class="spring_1.Car">
      <constructor-arg value="Baoma" type="java.lang.String"/>
      <!-- 如果字面值包含特殊字符，可以使用<!CDATA[]>包裹起来 -->
      <!-- 属性值也可以使用<value>子节点进行直接配置 -->
      <constructor-arg type="java.lang.String">
           <value> <![CDATA[<ShangHai^>]]> </value>
      </constructor-arg>
      <constructor-arg type="int" value=240/>
</bean>
```



### null值和级联属性

可使用专用的`<null/>`元素标签为Bean的字符串或其他对象类型的属性注入null值。同时Spring支持级联属性的配置。 

```xml
<bean id="person2" class="spring_1.Person">
    <constructor-arg value="BX"/>
    <constructor-arg value="24"/>
    <!--正常引用: <constructor-arg ref="car"/>   -->
    <!--测试赋值null:<constructor-arg><null/></constructor-arg>    -->
    <constructor-arg ref="car"/>
    <!-- 为级联属性赋值，注意，属性要先初始化后才可以为属性赋值，否则有异常，和Struts2不同 -->
    <property name="car.maxSpeed" value="250"/>
</bean>
```



### 集合(含数组)

- 该集合属性是仅使用一次的集合bean，即**内部bean**：

  - 在Spring中可以通过一组内置的 xml 标签（如：`<list>`，`<set>`或<`map>`）来配置集合属性；

  - 配置**`java.util.List`**类型的属性，需指定`<list>`标签，在标签里包含一些元素，这些标签可以通过`<value>`指定简单的常量值，通过`<ref>`指定对其他 Bean 的引用，通过`<bean>`指定内置Bean定义，通过`<null/>`指定空元素，甚至可以内嵌其他集合。

  - **数组**的定义和List一样，都是用`<list>`。 

  - 配置**`java.util.Set`**需要使用`<set>`标签，定义元素的方法和List一样。 

    ```xml
    <bean id="person" class="collections.Person">
       <property name="name" value="古河渚"/>
       <property name="age" value="17"/>
       <property name="cars">
          <list>   <!-- 使用list节点为List类型的属性赋值 -->
             <ref bean="car"/>
             <ref bean="car2"/>
             <bean class="spring.Car">
               <constructor-arg value="Ford"/>          
               <constructor-arg value="Changan"/>           
               <constructor-arg value="200000" type="double"/>    
             </bean>
           </list>
       </property>
    </bean>
    ```

  - **`java.util.Map`**通过`<map>`标签定义，`<map>`标签里可以使用多个`<entry>`作为子标签，每个条目包含一个键和一个值。 	必须在`<key>`标签里定义键 。

    - 键和值的类型没有限制，可以自由地为它们指定`<value>，<ref>，<bean>或<null>`元素。 
    - 可将Map的键和值作为`<entry>`的属性定义：简单常量使用key和value来定义；Bean引用通过key-ref和value-ref属性定义。 

    ```xml
    <bean id="newPerson" class="collections.NewPerson">
        <property name="name" value="冈崎汐"/>
        <property name="age" value="14"/>
        <property name="cars">
            <map>   <!-- 使用map节点和entry子节点配置Map类型的属性 -->
                <entry key="AA" value-ref="car"/>
                <entry key="BB" value-ref="car2"/>
            </map>
        </property>
    </bean>
    ```

- 外部集合bean，作为一个单独的bean，可以在其他bean之间共享。见本节最开始部分。



### properties属性的注入

使用`<props>`定义**`java.util.Properties`**，该标签使用多个`<prop>`作为子标签。每个`<prop>`标签必须定义key属性。 

```xml
<!-- 配置Properties属性值 -->
<bean id="dataSource" class="collections.DataSource">
  <property name="properties">
  <!-- 使用props节点和prop子节点为Properties属性赋值 -->
     <props>
       <prop key="user">root</prop>
       <prop key="password">132128</prop>
       <prop key="jdbcURL">jdbc:mysql:///localhost:3306//testjdbc</prop>
       <prop key="driverClass">com.mysql.jdbc.Driver</prop>
     </props>
   </property>
</bean>
```



# 1.4 混合配置

Spring提供的三种配置方案是可以混合使用的。

对于两种显式配置方式，通常创建一个根配置（root configuration），以此来组合其他的配置类/文件。在根配置中开启组件扫描，然后使用隐式配置。



## 配置类为主

使用时，仅使用组合类即可。

- 配置类+配置类：

  - 配置类由于配置了很多bean而变得笨重时，可以将其拆分为多个配置类，然后再创建一个新的配置类（也可以使用拆分后的某一个配置类来组合），用于组合其他的配置类。

  - **使用`@Import`注解导入其他配置类**：

    ```java
    @Configuration
    @Import({CDPlayerConfig.class,CDConfig.class})
    public class Config {   }
    ```

- 配置类+配置文件：

  - **使用`ImportResource`注解来向配置类中导入配置文件**：

    ```java
    @Configuration
    @Import(CDPlayerConfig.class)
    @ImportResource("classpath:soundsystem.xml")
    public class Config {    }
    ```

- 配置类+外部属性文件properties

  - **使用`@PropertySource`注解来引入外部配置文件**：

    ```java
    //使用@PropertySource找到外部配置文件，将其k/v保存到运行的环境变量中。/代表从类路径地根目录开始
    @PropertySource(value = {"classpath:/person.properties"})
    @Configuration
    public class MainConfigOfPropertyValue {
        @Bean
        public Person person(){
            return new Person();
        }
    }
    ```



## 配置文件为主

使用时，仅使用组合的配置文件即可。

- 配置文件+配置类

  - 向配置文件中导入配置类，只需要将配置类当作普通bean一样声明即可：

    ```xml
    <bean class="chapter2.config.CDPlayerConfig"/>
    ```

- 配置文件+配置文件

  - 使用`<import>`元素来引入其他配置文件

    ```xml
    <import resource="cdConfig.xml">
    ```

- 配置文件+外部属性文件properties

  - Spring提供了一个`PropertyPlaceholderConfigurer`的`BeanFactory`后置处理器（见后面），这个处理器允许用户将Bean配置的部分内容外移到属性文件中，可以在Bean配置文件里使用形式为`${var}`的变量，`PropertyPlaceholderConfigurer`从属性文件里加载属性，并使用这些属性来替换变量。

  - Spring还允许在属性文件中使用`${propName}`，以实现属性之间的相互引用。

  - Spring2.5后，可通过`<context:property-placeholder>`元素：在`<bean>`中添加context Schema定义在配置文件中配置。

    ```xml
    <!-- 导入属性文件 -->
    <context:property-placeholder location="classpath:db.properties"/>
    
    <bean id="dataSource" class="com.mchange.v2.c3p0.ComboPooledDataSource">
        <!-- 使用外部属性文件的属性 -->
        <property name="user" value="${user}"></property>
        <property name="password" value="${password}"></property>
        <property name="driverClass" value="${driverclass}"></property>
        <property name="jdbcUrl" value="${jdbcurl}"></property>
    </bean>
    ```



# 1.5 Environment

完成配置后，启动程序，所有的配置都会被加载到Spring的`Environment`中，可以从中检索属性。

```java
@Autowired
private Environment environment;
```

- `Environment`的方法：
  - 获取指定属性：
    - `String getProperty(String key)`：获取指定属性
    - `String getProperty(String key,String defaultValue)`：如果指定属性不存在，返回默认值
    - `T getProperty(String key,Class<T> targetType)`：将第一个方法返回的String类型转换为指定类型返回
    - `T getProperty(String key, Class<T> targetType, T defaultValue)`：设置一个默认值
    - `String getRequiredProperty(String key)`：上面的四个方法如果没有指定默认值，且该属性没有定义，返回的是`null`，如果希望该属性必须定义，否则抛出异常，可以使用当前方法
    - `T getRequiredProperty(String key, Class<T> targetType)`：返回一个指定类型
  - 检查环境中是否有某个属性：`boolean containsProperty(String key)`
  - profile（见2.1）相关：
    - 获取激活的profiles：`String[] getActiveProfiles()`
    - 获取默认的profiles：`String[] getDefaultProfiles()`
    - 判断当前环境是否支持给定的profile：`boolean acceptsProfiles(String... profiles)`



# 1.6 @Import

`@Import`注解：快速导入组件，组件id默认是组件的全类名。案例：

```java
@Configuration
@Import({Color.class,Red.class,MyImportSelector.class,MyImportBeanDefinitionRegistrar.class} )
public class MainConfig02 {  }
```

- 通过`ImportSelector`的方式来获取Blue、Yellow

  ```java
    public class MyImportSelector implements ImportSelector {
        /**
         *
         * @param importingClassMetadata 当前标注@Import注解的类的所有注解信息
         * @return 导入到容器中的所有组件的全类名组成的数组
         */
        @Override
        public String[] selectImports(AnnotationMetadata importingClassMetadata) {
            //这里通过全类名的方式来构成数组
            return new String[]{"com.zzk.bean.Blue","com.zzk.bean.Yellow"};
        }
    }
  ```

- 通过`ImportBeanDefinitionRegistrar`的方式获取RainBow

  ```java
  public class MyImportBeanDefinitionRegistrar implements ImportBeanDefinitionRegistrar {
      /**
       * @param importingClassMetadata 当前类的注解信息
       * @param registry BeanDefinition注册类
       *                调用BeanDefinitionRegistry.registerBeanDefinition 方法可以把需要添加到容器的Bean手动注册进来
       */
      @Override
      public void registerBeanDefinitions(AnnotationMetadata importingClassMetadata, BeanDefinitionRegistry registry) {
          String ms1 = "com.zzk.bean.Red";
          String ms2 = "com.zzk.bean.Blue";
          boolean b1 = registry.containsBeanDefinition(ms1);
          boolean b2 = registry.containsBeanDefinition(ms2);
          if(b1 && b2){
              //指定Bean的定义信息（类型、作用域Scope。。。）
              RootBeanDefinition beanDefinition = new RootBeanDefinition(RainBow.class);
              //注册Bean，并指定Bean名
              registry.registerBeanDefinition("rainBow",beanDefinition);
          }
      }
  }
  ```