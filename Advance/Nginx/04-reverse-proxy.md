[反向代理](../../Base/02-HTTP/代理.md#反向代理)

这里将之前搭建的静态资源 Web 服务器作为上游服务，再搭建一个 Nginx 作为反向代理来演示如何使用 Nginx 搭建反向代理。可以由一台 Nginx 把请求按照负载均衡算法代理给多台上游服务器工作，这样就实现了水平扩展。

# Hello World

将静态资源 Web 服务器作为上游服务器，上游服务器一般对公网不提供访问：在 static.conf 中 listen 的端口前面加上 IP 地址：

```shell
server {
  listen 127.0.0.1:8080;  # 此时只能本机的进程来打开8080端口
  access_log logs/fms5cms.log;
  location / {
    alias /resources/;
    autoindex on;
    set $limit_rate 1m;
    index index.html;
  }
}
```

先停止 Nginx 服务：`./nginx -s stop`，再打开

设置反向代理：在 config 文件夹下创建 reverse.conf：

```shell
upstream local { # 上游服务器的列表，并将{}中所有的上游服务器命名为local
  server 127.0.0.1:8080;
}
server{
  server_name fms5cms.com; # 设置作为反向代理的服务器的域名，访问时使用http://fms5cms.com。Nginx接到这个域名请求的话，将该请求转移到下面location来处理
  listen 80;

  location / { # 对于所有的请求
    # 这里的配置都是ngx_http_proxy_module模块中的，详情见官网
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    # 后端的Web服务器可以通过X-Forwarded-For获取用户真实IP
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    # 将所有访问 fms5cms.com 的请求代理到上面配置的名为local的上游服务中
    proxy_pass http://local;
  }
}
```

注意，做反向代理的 Nginx 通常部署在公网 IP 所在的服务器上，没有购买域名的话，如在虚拟机中安装 Nginx，使用 Windows 下的浏览器访问的话，需要先修改浏览器所在机器上的 host 文件。

然后在浏览器中访问 http://fms5cms.com 发现可以访问到静态资源 Web 服务器的 index 页面！

# Cache

在 nginx.conf 配置文件的 http 指令块中设置缓存文件的目录：

```shell
proxy_cache_path /tmp/nginxcache levels=1:2 keys_zone=my_cache:10m max_size=10g inactive=60m use_temp_path=off;
# 缓存路径 /tmp/nginxcache，关键字放在命名为 my_cache 的 10M 大小的共享内存
```

然后在需要做缓存的 URL 路径下添加 proxy_cache， reverse.conf：

```shell
upstream local {
  server 127.0.0.1:8080;
}
server{
  server_name fms5cms.com;
  listen 80;

  location / {
    proxy_set_header Host $host;
    proxy_set_header X-Real-IP $remote_addr;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;

    proxy_cache my_cache; #参数为上面开辟的共享内存
    proxy_cache_key $host$uri$is_args$args; # 共享内存中的key设置
    proxy_cache_valid 200 304 302 1d; # 对于哪些响应不返回

    proxy_pass http://local; # 代理到上面配置的上游服务中
  }
}
```

第一次访问 http://fms5cms.com，反向代理的 Nginx 收到静态资源 Web 服务器的响应后就会把响应缓存到自己的文件系统中，下次再访问就直接从缓存中拿了。

# 调度策略

负载均衡的调度策略：

注意：这里的是**单机部署多个相同应用**，所以可以直接使用域名+端口来进行配置。如果是**多台服务器部署同一个应用**，要使用 IP 地址+端口，而不能使用域名！！！

- 轮询：
  - 优点：实现简单
  - 缺点：不考虑每台服务器处理能力

```shell
upstream www.fms5cms.com{
  server www.fms5cms.com:8080;
  server www.fms5cms.com:9080;
  server www.fms5cms.com:7080 down; # down表示当前server暂时不参与负载
}
```

- 权重：
  - 优点：考虑了服务器处理能力的不同

```shell
upstream www.fms5cms.com{ # weigth参数表示权值，权值越高被分配到的几率越大，默认为1
  server www.fms5cms.com:8080 weight=15;
  server www.fms5cms.com:9080 weight=10;
}
```

- IP Hash：
  - 优点：能实现同一个用户访问同一个服务器
  - 缺点：服务器请求(负载)不平均(完全依赖 IP hash 的结果)

```shell
upstream www.fms5cms.com{
  ip_hash; # 采用 IP hash策略
  server www.fms5cms.com:8080;
  server www.fms5cms.com:9080;
}
```

- 地址散列(URL Hash)：需要安装 Nginx 插件
  - 优点：能实现同一个服务访问同一个服务器
  - 缺点：根据 URL Hash 分配请求会不平均，请求频繁的 URL 会请求到同一个服务器上

```shell
upstream www.fms5cms.com{
  server www.fms5cms.com:8080;
  server www.fms5cms.com:9080;
  hash $request_uri;
}
```

- fair：需要安装 Nginx 插件
  - 特点：按后端服务器的响应时间来分配请求，响应时间短的优先分配

```shell
upstream www.fms5cms.com{
  server www.fms5cms.com:8080;
  server www.fms5cms.com:9080;
  fair;
}
```

- 最少连接：
  - 优点：使集群中各个服务器负载更加均匀
- 加权最少连接：
  - 在最少连接的基础上，为每台服务器加上权值。算法为(活动连接数\*256+非活动连接数)/权重，计算出来的值小的服务器优先被选择。

# HTTPS 反向代理配置

对安全性要求比较高的站点，可能会使用 HTTPS。基础：

- HTTPS 协议的固定端口号是 443，不同于 HTTP 的 80 端口
- SSL 标准需要引入安全证书，所以在 nginx.conf 中你需要指定证书和它对应的 key

```shell
server {
  listen       443 ssl;
  server_name  www.helloworld.com;

  # ssl证书文件位置(常见证书文件格式为：crt/pem)
  ssl_certificate      cert.pem;
  #ssl证书key位置
  ssl_certificate_key  cert.key;

  # ssl配置参数（选择性配置）
  ssl_session_cache    shared:SSL:1m;
  ssl_session_timeout  5m;
  # 数字签名，此处使用 MD5
  ssl_ciphers  HIGH:!aNULL:!MD5;
  ssl_prefer_server_ciphers  on;

  location / {
    root   /root;
    index  index.html index.htm;
  }
}
```
