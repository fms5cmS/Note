原理：【看给容器中注册了什么组件，这个组件什么时候工作，这个组件的功能是什么？】

# 概念

Spring在SSM中作为Bean工厂，用来管理Bean的声明周期和框架集成。在Spring中主要使用了工厂模式、单例模式、代理模式。

参考：[对IoC和DI的理解](https://mp.weixin.qq.com/s/oFTSQh9bnzrfOhqY0TvV8w)

- ==IoC==（Inversion of Control 控制反转）：这一设计思想是反转资源获取的方向。
  - 传统Java程序中，Bean的属性通过构造器传参或 `setXxx()` 方法来传入，程序主动去创建依赖对象；
  - IoC是通过一个专门的容器来创建这些对象，即IoC容器控制了外部资源（不只是对象，也包括文件等）获取。因为由容器帮我们查找及注入依赖对象，对象只是被动的接收依赖对象，所以是反转。
  - `org.springframework.beans` 和 `org.springframework.context` 是 Spring IoC 容器的基础包。
- ==DI==（Dependency Injection 依赖注入）：IoC的另一种表述方式，由容器动态地将某个依赖关系注入到组件中。依赖注入的目的是为了提升组件重用的频率，并为系统搭建一个灵活、可扩展的平台。
  - 由于应用程序需要 IoC 容器来提供对象所需的外部资源，所以应用程序依赖于IoC容器；
  - IoC 容器为应用程序注入对象所需的外部资源（包括对象、资源、常量数据等）。
- ==AOP==：面向切面编程，扩展功能不是通过修改代码实现。（切的是方法而不是类）
  - 就是将代码切入到一个类的指定一个方法的指定位置（方法前 或 方法后）
  - 典型应用：日志打印、性能统计、异常处理。



# Spring容器

==Spring容器==：IoC容器实例化后，读取配置文件/类来创建 Bean 实例，之后才能从IoC容器中获取Bean实例并使用。

- Spring 提供了两种类型的 IoC 容器实现：
  1. `org.springframework.beans.factory.BeanFactory`接口：IoC容器的基本实现；
  2. ==`org.springframework.context.ApplicationContext`==接口：提供了更多高级特性，基于`BeanFactory`构建。代表了 Spring IoC 容器，且负责实例化、配置、聚合 beans。 下面是该接口的类：
     - `ClassPathXmlApplicationContext`类：从类路径下的XML配置文件中加载配置；
     - `FileSystemXmlApplicationContext`类：从文件系统中的XML配置文件中加载配置；
     - `AnnotationConfigApplicationContext`类：从配置类中加载配置；
     - `WebApplicationContext`接口：Web应用的IoC容器，允许从相对于Web根目录的路径中完成初始化工作。 
       - `AnnotationConfigWebApplicationContext`类：从配置类中加载配置；
       - `XmlWebApplicationContext`类：从Web应用下的XML配置文件中加载配置。
     - `ConfigurableApplicationContext` 接口扩展于`ApplicationContext `，新增两个主要方法：`refresh()`和`close()`，让`ApplicationContext`具有启动、刷新和关闭上下文的能力。



使用：

1. 创建 IoC 容器

   ```java
   //传入配置文件来获取 IoC 容器
   ApplicationContext context = new ClassPathXmlApplicationContext("spring-config.xml");
   //也可以根据配置类来获取 IoC 容器
   AnnotationConfigApplicationContext applicationContext = new AnnotationConfigApplicationContext(MainConfig01.class);
   ```

2. 获取配置文件中配置的 Bean

   ```java
   Person person = context.getBean(Person.class); //也可以通过配置的bean的id获取
   ```
