Nginx ("engine x") 是一个开源、高性能、可靠的 HTTP 服务器和反向代理服务器，也是一个 IMAP、POP3、SMTP 代理服务器。

> 其他常见的 HTTP 服务：HTTPD——Apache 基金会；IIS——微软；GWS——Google

Nginx 解决了服务器的 C10K(一秒之内连接客户端的数目为 10k 即 1 万)问题。它的设计不像传统的服务器那样使用线程处理请求，而是一个更加高级的事件驱动机制，这是一种异步事件驱动结构。

Nginx 可以作为静态页面的 web 服务器，同时还支持 CGI 协议的动态语言，比如 perl、php 等。但是不支持 java。

- Nginx 主要有三个**应用场景**：
  1. 静态资源服务：通过本地文件系统提供服务
  2. 反向代理服务：
     - Nginx 的强大性能
     - 缓存
     - 负载均衡
  3. API 服务：OpenResty
     - 由 Nginx 直接访问数据库或缓存服务或应用服务，利用 Nginx 强大的并发性能，实现如 Web 防火墙这样复杂的业务功能
- Nginx 的优点：
  - 高并发、高性能
  - 可扩展性好
  - 高可靠性
  - 热部署
  - IO 多路复用 epoll：
    - 多个描述符的 I/O 操作都能在一个线程内并发交替地顺序完成。复用指复用同一个线程；
    - IO 多路复用地实现方式：
      - select：能监视文件描述符地数量存在最大限制；线性扫描效率低下
      - poll
      - epoll：最大连接无限制，效率更高
  - 轻量级：功能模块少、代码模块化

# Linux 下安装

- 前置工作

```shell
# 关闭 iptables：
iptables -L  #查看是否有iptables规则
iptables -F  #关闭

# 停用 selinux：
getenforce  #查看
setenforce 0 #关闭

# 安装依赖
# 安装gcc（可输入gcc -v查询版本信息看系统是否自带安装）
yum install gcc
# 安装pcre
yum install pcre-devel
# 安装zlib
yum install zlib zlib-level
# 安装openssl，如需支持ssl才需安装openssl
yum install openssl openssl-devel
# 以上依赖安装可综合写成：
yum -y install gcc zlib zlib-devel pcre-devel openssl openssl-devel

# 安装工具
yum -y install wget httpd-tools
```

- 安装 Nginx

```shell
# 下载，官网 http://nginx.org/
wget http://nginx.org/download/nginx-1.14.2.tar.gz
# 解压
tar -zxvf nginx-1.14.2.tar.gz
# 进入解压后的目录中
cd nginx-1.14.2
# 安装
  # ./configure --prefix=/home/fms5cms/nginx  可指定安装目录
  # 不指定路径的话，默认安装在 /usr/local/nginx 下
  # 执行完成后，会在安装目录的 objs 目录下生成一些中间文件
  # 主要有一个 ngx_modules.c 文件，该文件决定了接下来的编译过程中，有哪些模块会被编译进 Nginx
./configure
# 编译
  # 运行完成后在 objs 下会生成大量的中间文件和最终运行的二进制文件
  # c 语言编译的所有中间文件都放在 objs/src 下
  # 如果使用了动态模块，动态模块编译生成的动态文件也会在 objs 下
  # 如果升级 Nginx 版本，不能执行 make install，需要从 objs 下把目标文件 nginx 拷贝到安装目录
make install
```

安装完成后，可以将 安装目录/sbin 这个写入 /etc/profile 文件中的 PATH。

# Nginx 的组成

- Nginx 二进制可执行文件：由各模块源码编译出的文件
- Nginx.conf 配置文件：控制 Nginx 的行为
- access.log 访问日志：记录每一条 HTTP 请求信息
- error.log 错误日志：定位问题

# Nginx 架构

[官网介绍](http://www.aosabook.org/en/nginx.html)

Nginx 启动时，会生成两种类型的进程，一个是主进程 （ Master ）， 一个或多个工作进程 （ Worker ）。主进程并不处理网络请求，主要负责调度工作进程、加载配置 、 启动工作进程及非停升级。

Nginx 的 Worker 进程，包括核心和功能性模块 ，核心模块负责维持一个运行循环（ run-loop ），执行网络请求处理的不同阶段的模块功能，比如：存储读写、内容传输 、网络读写 、外出过滤 ，以及将请求发往上游服务器等。

![](../../images/Nginx-architecture.png)

启动 Nginx 后，使用 `ps -ef|grep nginx` 可发现至少有两个 Nginx 进程。

# 模块化设计

模块化的设计是 Nginx 的架构基础。 Nginx 服务器被分解为多个模块 ，每个模块就是一个功能模块 ，只负责自身的功能，模块之间严格遵循 “高内聚，低耦合” 的原则。

- 核心模块（ngx_core、ngx_errlog、ngx_conf、ngx_events、ngx_event、ngx_rpoll、ngx_regex）：
  - 提供错误日志记录 、配置文件解析、事件驱动机制、进程管理等核心功能。
- 标准 HTTP 模块：
  - 提供 HTTP 协议解析相关的功能，比如： 端口配置 、 网页编码设置 、 HTTP 响应头设置 等等。
- 可选 HTTP 模块：
  - 主要用于 扩展 标准的 HTTP 功能，让 Nginx 能处理一些特殊的服务，如： Flash 多媒体传输 、解析 GeoIP 请求、 网络传输压缩 、 安全协议 SSL 支持等。
- 邮件服务模块：
  - 用于支持 Nginx 的 邮件服务 ，包括对 POP3 协议、 IMAP 协议和 SMTP 协议的支持。
- 第三方模块：
  - 扩展 Nginx 服务器应用，完成开发者自定义功能，比如： Json 支持、 Lua 支持等。

# Nginx 事件驱动模型

在 Nginx 的异步非阻塞机制中，worker 进程在调用 IO 后，就去处理其他的请求，当 IO 调用返回后，会通知该 worker 进程 。对于这样的系统调用，主要使用 Nginx 服务器的事件驱动模型来实现。

Nginx 的 事件驱动模型由事件发送器、事件收集器和事件处理器三部分基本单元组成：

- 事件发送器：负责将 IO 事件发送到事件处理器 ；
- 事件收集器：负责收集 Worker 进程的各种 IO 请求；
- 事件处理器：负责各种事件的响应工作 。

事件发送器将每个请求放入一个 待处理事件列表 ，使用非阻塞 I/O 方式调用 事件处理器来处理该请求。其处理方式称为 “多路 IO 复用方法” ，常见的包括以下三种： select 模型、 poll 模型、 epoll 模型。
