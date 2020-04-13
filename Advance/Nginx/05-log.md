可以使用 access_log 来分析定位问题、分析用户的的数据，但无法实时分析 access_log，使用工具 [GoAccess](https://goaccess.io/) 可以以图形化的方式通过 websocket 协议将 access_log 的变化反映到浏览器中，方便分析问题。

安装 GoAccess：

```shell
# 安装依赖
yum -y install glib2 glib2-devel ncurses ncurses-devel GeoIP GeoIP-devel
# 安装GoAccess
wget https://tar.goaccess.io/goaccess-1.3.tar.gz
tar -xzvf goaccess-1.3.tar.gz
cd goaccess-1.3/
./configure --enable-utf8 --enable-geoip=legacy
make
make install
# 安装完成，查看版本
goaccess -V
```

Nginx 的 access_log 很灵活，我们可以自己向其中添加不同的各模块的内置变量，

当 access_log 是默认配置时，可以使用 log-format=COMBINED 格式，当我们修改了 access_log 的格式后，可以在 log-format 中重新定义我们所添加的格式。

GoAccess 会使用`-o`参数生成一个新的 HTML 文件，把当前 access_log 中的内容以 HTML 图表的方式展示出来，而当 access_log 变迁时，GoAccess 会新起一个 websocket 进程通过端口的方式把新的 access_log 消息推送到客户端。

```shell
 # 对 fms5cms.log 日志文件，将其输出到../html/report.html文件中，使用实时更新页面的方式--real-time-html，并指定时间、日期、日志格式
goaccess fms5cms.log -o ../html/report.html --real-time-html --time-format='%H:%M:%S' --date-format='%d/%b/%Y' --log-format=COMBINED
```

然后 WebSocket 准备接收来自客户的连接，当我们访问 report.html 时，会向该进程发起连接，由该进程向我们推送最新的 access_log 变更。

修改静态资源 Web 服务器的 tesstatict.conf 文件：

```shell
server {
  listen 8080;
  access_log logs/fms5cms.log;
  location /report.html { #每当访问report.html时将其重定向到/web/report.html
    alias /web/report.html;
  }
  location / {
    alias /resources/;
    autoindex on;
    set $limit_rate 1k;  #限制 Nginx 向客户端发送响应的速度，每秒传输 1kB 响应到客户端
    index index.html;
  }
}
```
