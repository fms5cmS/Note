# Logback

| 日志门面（日志的抽象层）                                     | 日志实现                                             |
| ------------------------------------------------------------ | ---------------------------------------------------- |
| ~~JCL（Jakarta Commons Logging）~~、**SLF4j（Simple Logging Facade for Java）**、~~jboss-logging~~ | Log4j、JUL（java.util.logging）、Log4j2、**Logback** |

使用：左边选一个门面（抽象层），右边选一个实现；

日志门面：SLF4j

日志实现：Logback。（SLF4j、Log4j、Logback出自同一人；Log4j2很好，但很多框架还没有兼容）

LogBack是一个日志框架，它是Log4j作者Ceki的又一个日志组件。LogBack是Log4j的改良版本，拥有更多的特性，同时也带来很大性能提升。

# 1.导入

Logback分为三个模块：

- logback-core：提供Logback核心功能，是另外两个组件的基础；
- logback-classic：实现了 SLF4j 的API，如果要配合 SLF4j  使用，需要导入此包；
- logback-access：为了继承Servlet环境而准备，可提供HTTP-access的日志接口。

需要引入的模块：

```xml
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-api</artifactId>
    <version>1.7.25</version>
</dependency>
<!--注意，这里 slf4j-simple 的scope不能设置为test-->
<dependency>
    <groupId>org.slf4j</groupId>
    <artifactId>slf4j-simple</artifactId>
    <version>1.7.25</version>
</dependency>
<!--由于该包依赖于 logback-core包，所以没有显式地配置其依赖-->
<dependency>
    <groupId>ch.qos.logback</groupId>
    <artifactId>logback-classic</artifactId>
    <version>1.2.3</version>
</dependency>
```



# 2.配置

当使用logback时，logback通过如下几种方式读取配置启动：

1. 从系统配置文件System Propertis中寻找logback.configurationfile对应的value    （？不确定）
2. 在工程的classpath下寻找logback.groovy（logback支持通过groovy来配置，更直观）
3. 在classpath下寻找logback-test.xml
4. 在classpath下寻找logback.xml
5. 没有找到任何配置，使用默认配置将日志打印到控制台（调用ch.qos.logback.classic.BasicConfigurator的configure方法，构造一个ConsoleAppender用于向控制台输出日志，默认日志输出格式为”%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} – %msg%n”）。

**配置文件**

## `<configuration>`标签

- scan 属性：取值 true/false。默认取值为true，即配置文件如果发生改变，将会被重新加载；
- scanPeriod 属性： 设置监测配置文件是否有修改的时间间隔，默认时间间隔为1分钟。如果要自定义地话，默认单位为毫秒。需要配合`scan="true"`来一起使用；
- debug 属性：取值 true/false。默认为false。当取值为true时，将打印出logback内部日志信息，实时查看logback运行状态。

三个子标签：

### `<root>`标签

- level 属性：配置日志级别。可以调整输出的日志级别，==日志就只会在这个级别及以后的高级别生效==。

  ==日志级别（从低到高）：trace < debug < info < warn < error==

- `<appender>`子标签：用来配置日志输出地格式。

- `<appender-ref>`子标签：用来引用已经配置的 appender。

```xml
    <root>
        <level value="INFO" /> 	<!--这里将level属性设置为子节点了-->
        <appender-ref ref="STDOUT" />
    </root>
```

### `<logger>`标签

指定哪一些类或者包，使用某个appender配置，以及这些日志的level。

- level 属性：日志打印的级别设置，若不设置会默认获取上一级的level属性；
- name 属性：用来指定该logger约束的一个类或者包，即该logger只会打印类或包路径下的日志；
- additivity 属性：取值 true/false。默认为true，将此loger的打印信息向上级传递，一直会到root上。
- `<appender>`子标签：用来配置日志输出地格式。
- `<appender-ref>`子标签：用来引用已经配置的 appender。

```xml
<root level = "warn">
    <appender-ref ref = "appender1" />
</root>

<logger name = "java.lang" additivity = "true">
</logger>

<logger name = "java" level = "debug" additivity = "false">
    <appender-ref ref = "appender2" />
</logger>
```

如果按上述配置时，有一个java.lang包下的类异常时，会已经以下几步：

1. 首先会先找到name = “java.lang”的logger（下面简称langLogger），但langLogger没有配置level；
2. 默认获取上一级即name = “java”（下面简称javaLogger）的日志打印级别即debug，即langLogger可以输出debug及以上的日志；
3. langLogger没有配置日志输出格式appender，所以无输出，additivity配置为true即向上级javaLogger传输日志信息；
4. javaLogger根据level和appender，按appender2日志格式输出大于等于debug的日志；
5. javaLogger的additivity为false，即父节点root无法获取到日志，日志打印结束

### `<appender>`标签

	用于指定日志输出的目的地，目的地可以是控制台、文件、远程套接字服务器、 MySQL、 PostreSQL、Oracle和其他数据库、 JMS和远程UNIX Syslog守护进程等。

- name 属性：指定该 appender 的名称，便于引用。

- class 属性：指定该 appender 的全限定名，指定打印日志的实现。

- `<encoder>`子标签：输出日志的格式化。

  - `<charset>`属性：指定输出的编码

  - `<pattern>`子标签：规定格式

    ```xml
     <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern> 
     
    	%d 表示日期时间
        %thread 表示线程名
        %-5level 表示级别从左显示5个字符宽度
        %logger{50} 表示logger名字最长50个字符，否则按照句点分隔
        %msg 表示日志消息
        %n 是换行符
    ```

- `<filter>`子标签：过滤器，满足什么条件的日志会被持久化纪录

  - class 属性：指定使用的过滤器的全类名。下面举了三个例子

    - ch.qos.logback.core.filter.EvaluatorFilter  根据一些特定的值进行过滤。

      ```xml
      <appender name="appender" class="ch.qos.logback.core.rolling.RollingFileAppender">
          <file>${log.dir}/${projectname}/rollingAppender.log</file>
          <filter class="ch.qos.logback.core.filter.EvaluatorFilter">
              <evaluator>
                  <!-- 过滤掉所有日志消息中不包含"Exception"字符串的日志-->
                  <expression>return message.contains("Exception");</expression>
              </evaluator>
              <OnMatch>ACCEPT</OnMatch>
              <OnMismatch>DENY</OnMismatch>
          </filter>
      </appender>
      ```

    - ch.qos.logback.classic.filter.ThresholdFilter  根据特定的日志级别阈值过滤。对小于level的日志进行过滤，只打印level以上级别的日志

      ```xml
      <appender>
          <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
              <level>WARN</level>
          </filter>
      </appender>
      ```

    - ch.qos.logback.classic.filter.LevelFilter 只打印特定级别的日志

      ```xml
      <appender>
          <filter class="ch.qos.logback.classic.filter.LevelFilter">
              <level>WARN</level>
              <onMatch>ACCEPT</onMatch>
              <onMismatch>DENY</onMismatch>
          </filter>
      </appender>
      只会打印WARN级别的日志，因为在filter中对匹配到WARN级别时做了ACCEPT（接受），对未匹配到WARN级别时做了DENY（拒绝）
      ```



## 常用Appender

- **ch.qos.logback.core.ConsoleAppender 将日志打印到控制台**。默认使用 System.out。

  - 可以使用`<target>`子标签来控制输出到控制台时使用的是 System.out还是System.err。

  ```xml
  <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">	
      <target>System.out</target>
      <encoder charset="UTF-8">
          <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
      </encoder>
  </appender>
  ```

- **ch.qos.logback.core.FileAppender 将日志输出为文件**

  - 可以使用`<file>`子标签来决定输出的文件地址及文件名；
  - 可以使用`<append>`子标签：决定新的日志如何写入，默认为true。
    - 如果为true，末尾追加。
    - 如果为false，清空现有文件，rollingFileAppender也支持

  ```xml
  <appender name="fileAppender" class="ch.qos.logback.core.FileAppender">
      <file>${log.dir}/${projectname}/fileAppender.log</file>
      <encoder>
          <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
      </encoder>
      <append>true</append>
  </appender>
  ```

- **ch.qos.logback.core.rolling.RollingFileAppender 滚动记录日志**。先将日志记录到指定的文件中，当符合一定条件后再将日志迁移到其他文件

  - `<rollingPolicy> `子标签：滚动条件，达到什么条件转换纪录的文件

    ```xml
        <!--rolling的条件，达到什么条件转换纪录的文件，以下是按照时间，每日生成一个文件-->
        <!--还有一些其他的策略，包括文件大小等-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--转换后的日志名。是必要节点，包含文件名及”%d”转换符，”%d”可以包含一个Java.text.SimpleDateFormat指定的时间格式，如果直接使用%d那么格式为yyyy-MM-dd-->
            <fileNamePattern>${log.dir}/${projectname}/rollingAppender-%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <!--最大保存的文件数量，本例中是基于时间的滚动，所以是最多保存30天-->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
    ```

- **ch.qos.logback.classic.AsyncAppender 异步输出**

  ```xml
      <!-- 异步输出 -->
      <appender name ="asyncAppender" class= "ch.qos.logback.classic.AsyncAppender">
          <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
          <discardingThreshold >0</discardingThreshold>
          <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
          <queueSize>2048</queueSize>
          <!-- 添加附加的appender,最多只能添加一个 -->
          <appender-ref ref ="rollingAppender"/>
          <includeCallerData>true</includeCallerData>
      </appender>
  ```



## XML文件示例

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!-- scan:当此属性设置为true时，配置文件如果发生改变，将会被重新加载，默认值为true。-->
<!--scanPeriod: 设置监测配置文件是否有修改的时间间隔，如果没有给出时间单位，默认单位是毫秒。当scan为true时，此属性生效。默认的时间间隔为1分钟。-->
<!--debug: 当此属性设置为true时，将打印出logback内部日志信息，实时查看logback运行状态。默认值为false-->
<configuration debug="false" scan="true">
    <!--contextName:工程名-->
    <contextName>LogbackDemo</contextName>
    <!--property:变量值，在后续配置中以${log.dir}的方式引用-->
    <property name="log.dir" value="E:\logs\" />
    <property name="projectname" value="demo" />

    <!--对于appender标签，name是appender的名称，class是全限定名。以下每个appender是输入到不同位置，具体看class基本就能看出来-->
    <!--encoder标签就是输出日志的格式化-->
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">\
        <!--target：输出到控制台时，是使用System.out还是System.err-->
        <target>System.out</target>
        <encoder charset="UTF-8">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>

    <appender name="fileAppender" class="ch.qos.logback.core.FileAppender">
        <file>${log.dir}/${projectname}/fileAppender.log</file>
        <encoder>
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
        <!--append:如果为true，末尾追加。如果为false，清空现有文件，rollingFileAppender也支持，默认为true-->
        <append>true</append>
    </appender>

    <!--RollingFileAppender 滚动记录文件，先将日志记录到指定的文件，当符合某个条件的时候，将日志记录到其他文件-->
    <appender name="rollingAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <file>${log.dir}/${projectname}/rollingAppender.log</file>
        <!--过滤器，满足什么条件的日志会被持久化纪录。此处用到是ThresholdFilter，还有一些其他的过滤，EvaluatorFilter，根据一些特定的值进行过滤-->
        <!--<filter class="ch.qos.logback.core.filter.EvaluatorFilter">-->
            <!--<evaluator>-->
                <!--<!– 过滤掉所有日志消息中不包含"Exception"字符串的日志 –>-->
                <!--<expression>return message.contains("Exception");</expression>-->
            <!--</evaluator>-->
            <!--<OnMatch>ACCEPT</OnMatch>-->
            <!--<OnMismatch>DENY</OnMismatch>-->
        <!--</filter>-->
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>INFO</level>
        </filter>
        <!--rolling的条件，达到什么条件转换纪录的文件，以下是按照时间，每日生成一个文件-->
        <!--还有一些其他的策略，包括文件大小等-->
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <!--转换后的日志名-->
            <fileNamePattern>${log.dir}/${projectname}/rollingAppender-%d{yyyy-MM-dd}.log
            </fileNamePattern>
            <!--最多保存30天-->
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <encoder charset="UTF-8">
            <pattern>%d{yyyy-MM-dd HH:mm:ss.SSS} [%thread] %-5level %logger{50} - %msg%n</pattern>
        </encoder>
    </appender>

    <!-- 异步输出 -->
    <appender name ="asyncAppender" class= "ch.qos.logback.classic.AsyncAppender">
        <!-- 不丢失日志.默认的,如果队列的80%已满,则会丢弃TRACT、DEBUG、INFO级别的日志 -->
        <discardingThreshold >0</discardingThreshold>
        <!-- 更改默认的队列的深度,该值会影响性能.默认值为256 -->
        <queueSize>2048</queueSize>
        <!-- 添加附加的appender,最多只能添加一个 -->
        <appender-ref ref ="rollingAppender"/>
        <includeCallerData>true</includeCallerData>
    </appender>

    <!--logger指定哪一些类或者包，使用某个appender配置，以及这些日志的level，TRACE < DEBUG < INFO < WARN < ERROR-->
    <!--logger中有个属性additivity: 默认为true，将此loger的打印信息向上级传递，一直会到root上-->
    <logger name="org.apache">
        <level value="INFO" />
        <appender-ref ref="fileAppender" />
        <appender-ref ref="rollingAppender" />  
    </logger>

    <logger name="org.springframework">
        <level value="INFO" />
        <appender-ref ref="fileAppender" />
        <appender-ref ref="rollingAppender" />  
    </logger>

    <logger name="com.netflix">
        <level value="INFO" />
        <appender-ref ref="fileAppender" />
        <appender-ref ref="rollingAppender" />
    </logger>

    <logger name="asyncLog">
        <level value="INFO" />
        <appender-ref ref="fileAppender" />
        <appender-ref ref="rollingAppender" />
    </logger>
    <!--root 也是元素,但是它是根logger。只有一个level属性。因为已经被命名为为root-->
    <root>
        <level value="INFO" />
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```











