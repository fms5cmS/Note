# 配置语法

1. 配置文件由指令与指令块构成
2. 每条指令以 `;` 结尾，指令与参数间以空格分隔，一条指令可以有多个参数
3. 指令块以 `{}` 将多条指令组织在一起
4. `include` 语句允许组合多个配置文件以提升可维护性
5. 使用 `#` 添加注释
6. 使用 `$` 使用变量
7. 部分指令的参数支持正则

- 配置参数
  - 时间的单位：ms、s、m、h、d、w、M、y
  - 空间的单位：
    - 不加任何后缀即为 bytes
    - k / K：千字节
    - m / M：兆
    - g / G
- http 配置的指令块：
  - http：块内的所有内容由 http 模块来解析，其他模块无法解析
    - server：对应一个或一组域名
    - location：URL 表达式
    - upstream：上游服务，当 Nginx 需要与 Tomcat 等其他内网服务交互时使用

配置部分的笔记参考：[超实用的 Nginx 极简教程，覆盖了常用场景](https://mp.weixin.qq.com/s?__biz=MzUzMTA2NTU2Ng==&mid=2247486382&idx=1&sn=7ef8aacd15b7d78b84c31902a4bb1186&chksm=fa49741fcd3efd091f59cee08a2276bc81e70bb49959d01fe1f76c886b00acb6c910b670c52a&scene=0&pass_ticket=GxPiieA1J2Rm%2BDdrRZyO10oku1mLNAcFyPNnlkw09Xqx1iSs8GiQSymPtxG3h2wv#rd)

# 配置文件

安装目录/conf/nginx.conf

```shell
#运行用户
#user somebody;

#启动进程,通常设置成和cpu的数量相等
worker_processes  1;

#全局错误日志
#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#PID文件，记录当前启动的nginx的进程ID
pid        D:/Tools/nginx-1.10.1/logs/nginx.pid;

#工作模式及连接数上限
events {
  worker_connections 1024;    #单个后台worker process进程的最大并发链接数
}

#设定 http 服务器，利用它的反向代理功能提供负载均衡支持
http {
  #设定mime类型(邮件支持类型),类型由 mime.types 文件定义
  include       mime.types;
  default_type  application/octet-stream;

  #设定日志格式，并将其进行了命名(main)，因为我们可能对不同域名下做不同的日志记录，使用 log_format 配置好格式后通过 access_log 来设置日志记录在哪里
  log_format  main  '[$remote_addr] - [$remote_user] [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';
  # 将日志记录在logs/access.log中，日志格式采用命名为main的格式
  access_log    logs/access.log main;
  rewrite_log     on;

  #sendfile 指令指定 nginx 是否调用 sendfile 函数（zero copy 方式）来输出文件，对于普通应用，
  #必须设为 on,如果用来进行下载等应用磁盘IO重负载应用，可设置为 off，以平衡磁盘与网络I/O处理速度，降低系统的uptime.
  sendfile        on;
  #tcp_nopush     on;

  #连接超时时间
  keepalive_timeout  120;
  tcp_nodelay        on;

  #gzip压缩开关
  #gzip  on;

  #设定实际的服务器列表
  upstream zp_server1{
    server 127.0.0.1:8089;
  }

  #HTTP服务器
  server {
    listen       80; #监听端口号
    server_name  www.helloworld.com; #服务名,定义使用www.xx.com访问
    #charset utf-8; #字符集
    #access_log  logs/host.access.log  main;  通常每个 server 设置一个日志文件

    #location [=|~|~*|^~] /uri/ { … }
    # = 精确匹配
    # ~ 正则匹配，区分大小写
    # ~* 正则匹配，不区分大小写
    # ^~  关闭正则匹配

    #匹配原则：
    # 1、所有匹配分两个阶段，第一个叫普通匹配，第二个叫正则匹配。
    # 2、普通匹配，首先通过“=”来匹配完全精确的location
    #   2.1、 如果没有精确匹配到， 那么按照最大前缀匹配的原则，来匹配location
    #   2.2、 如果匹配到的location有^~,则以此location为匹配最终结果，如果没有那么会把匹配的结果暂存，继续进行正则匹配。
    # 3、正则匹配，依次从上到下匹配前缀是~或~*的location, 一旦匹配成功一次，则立刻以此location为准，不再向下继续进行正则匹配。
    # 4、如果正则匹配都不成功，则继续使用之前暂存的普通匹配成功的location.


    location / {  # 匹配任何查询，因为所有请求都以 / 开头。但正则表达式规则和长的块规则将被优先和查询匹配。
      root   html; # 定义服务器的默认网站根目录位置
      index  index.html index.htm; # 默认访问首页索引文件的名称
      proxy_pass http://myserver; # 反向代理路径（和upstream绑定），location 后面设置映射的路径
      proxy_connect_timeout 10; # 反向代理的超时时间
      proxy_redirect default;
    }

    location  /images/ {
      root images ;
    }

    location ^~ /images/jpg/ {  # 匹配任何以 /images/jpg/ 开头的任何查询并且停止搜索。任何正则表达式将不会被测试。
      root images/jpg/ ;
    }

    location ~*.(gif|jpg|jpeg)$ {
      root pic ; #所有静态文件直接读取硬盘
      #expires定义用户浏览器缓存的时间为3天，如果静态页面不常更新，可以设置更长，这样可以节省带宽和缓解服务器的压力
      expires 3d; #缓存3天
    }
    # 错误处理页面（可选择性配置）
    # error_page  404              /404.html;
    # redirect server error pages to the static page /50x.html
    error_page   500 502 503 504  /50x.html;
      location = /50x.html {
      root   html;
    }
  }
}
```
