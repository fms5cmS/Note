# 1.1 手动创建项目

功能：浏览器发送hello请求，服务器接受请求并处理，响应Hello World字符串；

**步骤一**、Maven 创建 jar 包工程

**步骤二**、导入依赖

```xml
<parent>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-parent</artifactId>
    <version>2.0.2.RELEASE</version>
    <relativePath/> <!-- 父项目定义了所有依赖的版本 -->
</parent>

<dependencies>
    <!--SpingBoot开发Web项目的起步依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
    <!--导入配置文件处理器，配置文件进行绑定就会有提示-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-configuration-processor</artifactId>
        <optional>true</optional>
    </dependency>
    <!--进行单元测试的起步依赖-->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>

    <build>
        <plugins>
            <plugin>
                <!--插件功能：打包过程中生成一个可执行的jar包-->
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
```

**步骤三**、编写一个主程序：启动SpringBoot应用 

```java
@SpringBootApplication //声明所标注的类是一个SpringBoot的配置类。
public class HelloWorldMainApplication {
    public static void main(String[] args) {
        //Spring 应用启动起来。这里传入的类必须是一个@SpringBootApplication标注的类
    	SpringApplication.run(HelloWorldMainApplication.class,args);
    }
}
```

**步骤四**、编写相关的 Controller、Service 业务逻辑

```java
@Controller
public class HelloController {
    @ResponseBody
    @RequestMapping("/hello")
    public String hello(){
        return "hello world!";
    }
}
```

**步骤五**、运行主程序测试 

**步骤六**、简化部署

 在maven 的 pom 文件中导入插件，然后将这个应用打成jar包，使用`java -jar jar文件`命令启动程序 。

```xml
<!--这个插件，可以将应用打包成一个可执行的jar包-->
<build>
    <plugins>
        <plugin>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
        </plugin>
    </plugins>
</build>
```





# 1.2 快速创建项目

IDE都支持使用 Spring Initializr 快速创建 SpringBoot 项目：选择需要的模块，向导会**联网**创建SpringBoot项目。 

1. Eclipse：使用STS创建Spring Starter 工程 ，然后选择模块。。。 
2. IDEA：选择 Spring Initializr，然后选择模块。。。 

默认生成的SpringBoot项目：

- 主程序已经生成好了，只需要写自己的逻辑 
- resources 文件夹中目录结构 
  - static 文件夹：保存所有的静态资源：js、css、images... 
  - templates 文件夹：保存所有的模板页面，SpringBoot默认jar包使用嵌入的Tomcat，默认不支持 JSP 页面，可以使用模板引擎（freemarker、thymeleaf） 
  - application.properties：SpringBoot 应用的配置文件，可以修改一些默认的配置 。（也可以自己创建application.yml配置文件。注意：**配置文件的名字是固定的**）
    - 如输入：`server.port = 8081`指定端口为8081 
- 测试类：可以在测试期间很方便地类似编码一样进行自动注入等容器的功能 
  - 注解`@SpringBootTest`声明这是一个SpringBoot单元测试 
  - 注解`@RunWith(SpringRunner.class)`声明该测试使用`SpringRunner`来运行 

```java
@RunWith(SpringRunner.class)
@SpringBootTest
public class Springboot01ApplicationTests {
    @Test
    public void contextLoads() { }
}
```



# 1.3 更换父工程

如果不想使用`spring-boot-starter-parent`作为父工程，而是使用自定义的父工程该怎么做呢？通过以下方式即可。

```xml
<dependencyManagement>
    <dependencies>
        <!-- Spring Boot的依赖关系 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-dependencies</artifactId>
            <version>${spring-boot.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<build>
    <plugins>
        <plugin>
            <!--插件功能：打包过程中生成一个可执行的jar包-->
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-maven-plugin</artifactId>
            <version>2.1.3.RELEASE</version>
            <executions>
            	<execution>
                	<goals>
                    	<goal>repackage</goal>
                    </goals>
                </execution>
            </executions>
        </plugin>
    </plugins>
</build>
```

