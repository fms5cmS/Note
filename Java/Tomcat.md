# 安装

在[官网](https://tomcat.apache.org/download-80.cgi)下载后，解压。解压目录：

- bin：可执行文件（Windows系统下：startup.bat、shutdown.bat；Linux系统下：后缀为sh）
- conf：配置文件（server.xml等）
- lib：tomcat依赖的jar包
- logs：日志文件（记录出错等信息）
- temp：临时文件
- webapps：可执行的项目（开发后的项目放入这里）
- work：存放JSP翻译后的java、及编译后的class文件

配置：可以通过阅读startup.bat了解启动顺序来配置

1. 必须先安装并配置JDK；
2. 配置CATALINA_HOME环境变量，指向 tomcat 解压后的目录，如：`D:\DevelopInstall\apache-tomcat-8.5.35`
3. 在Tomcat配置文件server.xml的`<Connector port="8080"..>`标签中加入`URIEncoding="UTF-8"`，可解决GET方式时地址栏中的乱码问题；
4. 如果想要在命令行输入`startup.bat`直接启动Tomcat的话，还需要在Path环境变量中增加指向Tomcat安装目录下bin目录。

关联 Tomcat 和 IDEA。

**虚拟路径**：将Web项目配置到 webapps 以外的目录。

- 方式一（需要重启Tomcat）：在 server.xml 中的`<Host>`标签中添加`<Context>`标签，标签属性：
  - docBase：实际路径
  - path：虚拟路径（绝对路径、相对路径（相对于webapps））
- 方式二（不用重启Tomcat）：在conf\Catalina\localhost 文件夹下创建文件：项目名.xml
  - 文件中写入方式一中的`<Context>`即可。
  - 如果文件名为 ROOT.xml （内容不变），则在浏览器中输入url时就不用再写项目名了。



# 注意

Tomcat 8.5后，Tomcat 解析 Cookie 发生改变，domain 属性必须以数字或字母开头，而不能以点开始，所以会报“An invalid domain was specified for this cookie”错误，为了使得 Tomcat 可以解析以点开头的 domain，必须修改 Tomca t目录下 /conf/content.xml 文件：在`<context></context>`标签中加入：

`<CookieProcessor className="org.apache.tomcat.util.http.LegacyCookieProcessor" />`

# Tomcat集群

- Tomcat集群可以
    - 提高服务的
        - 性能：实际的线上部署环境都会选择一台机器部署一个Tomcat。如果一台机器部署多个Tomcat，会存在共享瓶颈，如：使用同一个网卡、内存共用、磁盘I/O性能；
        - 并发能力：一台 Tomcat 的 Http 线程池是有限的(根据机器的性能)，自然的，多台Tomcat可承载Http线程就是一台的多倍；
        - 高可用性
    - 提高项目架构的横向扩展能力
        - 一台服务器通过不断升级内存、CPU、换为固态硬盘等，这些是纵向提高机器配置，来提高Tomcat所提供服务的性能



Tomcat集群实现原理：通过Nginx负载均衡进行请求转发。



# 单机部署多应用

CATALINA_BASE 和 TOMCAT_HOME可以不配置，仅配置CATALINA_HOME。



## Linux环境

1. 修改 /etc/profile 增加Tomcat环境变量(这里是一台机器部署两个Tomcat)，`source /etc/profile`使其生效。
```shell
export CATALINA_BASE=第一个Tomcat安装的目录
export CATALINA_HOME=第一个Tomcat安装的目录
export TOMCAT_HOME=第一个Tomcat安装的目录

export CATALINA_2_BASE=第二个Tomcat安装的目录
export CATALINA_2_HOME=第二个Tomcat安装的目录
export TOMCAT_2_HOME=第二个Tomcat安装的目录
```
2. 配置两个Tomcat
    - 第一个Tomcat不变
    - 第二个Tomcat：
        - 打开安装目录下bin/catalina.sh，找到`# OS specific support.  $var _must_ be set to either true or false.`，在这一行下面新增配置，保存
        ```shell
        export CATALINA_BASE=$CATALINA_2_BASE
        export CATALINA_HOME=$CATALINA_2_HOME
        ```
        - 打开安装目录下conf/server.xml，注意：三个端口都要修改！且多个Tomcat之间一定不能重复
            - `<Server port>`节点端口号修改
            - `<Connector port>`节点端口号8080修改，这是Tomcat的访问端口，注意，节点内的redirectPort不修改
            - `<Connector port>`节点端口号8009修改
    - 分别进入两个Tomcat的bin目录，启动Tomcat，检查两个Tomcat的启动日志
    - 分别访问两个Tomcat的访问路径，可以打开Tomcat部署的webapps的ROOT项目首页



## Windows环境

1. 在系统环境变量中增加Linux环境下/etc/profile中的变量；
2. 配置两个Tomcat：
    - 第一个不变；
    - 第二个：
        - 打开安装目录下bin/catalina.bat和startup.bat
            - 替换两个文件中:
                - CATALINA_BASE——>CATALINA_2_BASE
                - CATALINA_HOME——>CATALINA_2_HOME
        - 打开安装目录下conf/server.xml，修改三个端口(同Linux的操作)
    - 分别启动两个Tomcat，检查两个Tomcat的启动日志
    - 分别访问两个Tomcat的访问路径，可以打开Tomcat部署的webapps的ROOT项目首页



# 多机部署多应用

比单机多实例简单很多。每个机器部署一个Tomcat，端口号也不用修改；如果一个机器部署多个Tomcat，参照3.2。

注：多个服务器且每个服务器只安装一个Tomcat，要保证它们之间的网络是互通的，才可以集群。Nginx装在任意一台服务器上即可，也可把Nginx服务独立出来一台，也要和Tomcat的网络互通。

