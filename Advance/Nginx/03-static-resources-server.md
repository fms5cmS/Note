- 静态资源类型：非服务器动态运行产生的文件
  - 浏览器端渲染：HTML、CSS、JS
  - 图片：JPEG、GIF、PNG
  - 视频：FLV、MPEG
  - 文件：TXT 等任意下载文件

在主配置文件的 http 指令块中(与 server 指令块同级)加入：`include config/*.conf;`，在 conf 文件夹下创建 config 文件夹用于放置自己的配置(以 .conf 为后缀)，便于管理，创建一个 static.conf，下面的配置不再单独说明都是在这个文件中配置。

# Hello World

1. 放入静态资源文件夹。假设在根目录 / 下有 resources 的文件夹，里面有一个 a.txt 的文件；
2. 配置 Nginx 见下面
3. 在浏览器中访问 `http://192.168.*.*:8080/a.txt` 即可在浏览器中显示 a.txt 文件内容，访问 `http:192.168.*.*:8080/` 访问首页

```shell
server {
  listen 8080; #监听端口
  # 将日志记录在logs/fms5cms.log中，日志格式采用命名为main的格式
  access_log logs/fms5cms.log;
  #charset utf-8; #字符集

  location / {  #所有的请求
    # 需要指定 URL 的后缀要与文件目录的后缀一一对应，既可以使用 root 也可使用 alias
    # root 会把 URL 中的一些路径带到文件目录中，所以通常使用 alias
    alias /resources/;  # 服务器下的路径。Linux 下这个路径最后要带斜杠，Windows 下不带
    index index.html; #定义首页索引文件的名称
  }
}
```

在 nginx.conf 配置文件中 http 指令块中的 log_format 指令后面就使用了许多变量用于设置日志的格式，并将其进行了命名(main)，因为我们可能对不同域名下做不同的日志记录，通过 log_format 配置好格式后使用 access_log 来设置日志记录在哪里。

注意权限问题：为了保证文件的正常访问，**Nginx 需要文件的读权限和文件所有父目录的可执行权限**，所以这里在根目录下创建了 resources 文件夹，**对文件的父目录设置权限为 755，文件内容权限为 644**

# gzip 压缩

对于文本文件，是可以做 gzip 压缩的，做完 gzip 压缩后，传输的字节数会大幅度减少，所以通常会打开 gzip。在 nginx.conf 主配置文件中打开 gzip：

```shell
gzip on;
gzip_min_length 1;  # 小于1字节的内容就不再进行压缩
gzip_comp_level 2;  # 压缩级别
gzip_types text/plain application/x-javascript text/css application/xml text/javascript application/x-httpd-php image/jpeg image/gif image/png; #针对哪些文件类型才进行压缩
```

再次访问时可以从响应头中看到："Content-Encoding: gzip"

# autoindex

假设需要把静态资源文件夹的目录结构信息展示给用户由用户决定使用哪些文件，这一功能可使用 Nginx 提供的 ngx_http_autoindex_module 模块来完成，可以将处理以`/`结尾的请求时，显示对应目录的结构。

```shell
server {
    listen 8080;
    access_log logs/fms5cms.log;
    location / {
    	alias /resources/;
    	autoindex on;
    	index index.html;
    }
}
```

可以在 resources 目录下新增一个 img 文件夹，并放入几张图片，注意设置权限！

访问 `http://192.168.*.*:8080/` 就会显示结构。

# 限制响应

由于公网带宽有限，可能会让用户访问一些大文件时限制它的速度，以期望能分离出足够带宽给用户访问一些必要的小文件，如 CSS 等。可以使用 set 命令配合一些内置变量来实现。

```shell
server {
    listen 8080;
    access_log logs/fms5cms.log;
    location / {
    	alias /resources/;
    	autoindex on;
    	set $limit_rate 1k;  #限制 Nginx 向客户端发送响应的速度，每秒传输 1kB 响应到客户端
    	index index.html;
    }
}
```

这里设置为 1kB ，在访问时就会发现响应特别慢。

其他内置变量见官网 ngx_http_core_module 模块的[Embedded Variables](http://nginx.org/en/docs/http/ngx_http_core_module.html#variables)。不同的模块也有不同的内置变量！
