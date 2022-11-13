
https://gitee.com/geektime-geekbang/geektime-webprotocol
https://github.com/geektime-geekbang/geektime-webprotocol

HTTP 协议是一种**无状态的**、**应用层的**、**以请求/应答方式运行**的协议，它使用可扩展的语义和自描述消息格式，与基于网络的超文本信息系统灵活的互动。

HTTP 请求的状态：

- 有状态的请求，服务器端保存请求的相关信息，每个请求可以使用以前保留的请求相关信息
  - 服务器 session 机制使服务器保存请求的相关信息
  - cookie 使请求可以携带查询信息，与 session 配合完成有状态的请求
- 无状态的请求，服务器能够处理的所有信息都来自当前请求所携带的信息
  - 服务器不会保存 session 信息
  - 请求可以通过 cookie 携带

# 网络分层

- OSI(Open System Interconnect) 七层**概念模型**（从未被真正实现过！）
  - 应用层，用于解决业务问题
  - 表示层，负责把网络中的消息转换成应用层可以读取的消息
  - 会话层，负责建立会话、握手、维持连接关闭。只是一个概念，帮助理解 session 的概念
  - 传输层，解决进程与进程间的通信，TCP 协议还做了报文的可达性、流量控制
  - 网络层，IP 协议确保在广域网中可以将报文从一个主机发送到另一个主机
  - 数据链路层，局域网中通过 MAC 地址连接到相应的交换机、路由器等，然后就可以把报文发送到另一个主机了
  - 物理层，物理介质

每一层都利用其下层的服务，为其上层提供服务。第一层直接为第二层服务，第七层为模型外的用户服务。

![](../../images/osi.png)

对比 OSI 七层模型和 TCP/IP 模型：

![](../../images/osi-tcp_model.png)

工作在 OSI 二层（数据链路层）的网络设备为交换机，工作在 OSI 三层（网络层）的设备为路由器。

工作在 OSI 四层（传输层）的负载均衡器（如 LVS）可以将 Client 传过来的 TCP 报文以第二条 TCP 连接的方式转发给 Server，对于 UDP 会话也可以这样。四层负载均衡不会解析 TCP、UDP 上承载的应用层协议。

工作在 OSI 七层（应用层）的负载均衡器可以进行协议转换，示例：
1. Client 通过 HTTP/1.1 发送请求，假设 Server 不支持 HTTP/1.1，负载均衡器可以将请求转换为 HTTP/1.0 来连接 Server；
2. Client 通过 HTTP/1.1 + TLS 发送请求，假设 负载均衡到 Server 是基于企业内网的，其本身就是安全的，负载均衡器会剥离 TLS 协议，还可以将 HTTP/1.1 转换为更为高效的协议，如 uwsgi
3. Client 通过 HTTP/2 发送请求，假设 Server 不支持 HTTP/2，负载均衡器可以将请求转为 HTTP/1.1，如果 Server 是在外网部署的，还会加上 TLS

七层负载均衡可以解析出 header、body 等 message 中的各种信息，所以 WAF（Web Application Firewall）防火墙是可以天然放在七层负载均衡中的。

七层负载均衡通常还有缓存功能，因为它解析出了 header、body 等内容，就可以在七层负载均衡做一层缓存。

# HTTP 消息格式

基于 [ABNF](https://www.ietf.org/rfc/rfc5234.txt) 描述的 HTTP 协议格式：HTTP-message = `start-line *(header-field CRLF)CRLF [message-body]`

- start-line = `request-line / status-line`
  - request-line = `method SP request-target SP HTTP-version CRLF`
  - status-line = `HTTP-version SP status-code SP reason-phase CRLF`
- header-field = `field-name ":" OWS field-value OWS`
  - OWS = `*(SP/HTAB)`
  - field-name = `token`
  - field-value = `*(field-content / obs-fold)`
- message-body = `*OCTET`，0 或多个二进制字节流

示例：

```shell
$ telnet www.taohui.pub 80
Host 'www.taohui.pub' resolved to 116.62.160.193.
Connecting to 116.62.160.193:80...
Connection established.
To escape to local shell, press 'Ctrl+Alt+]'.
$ GET /wp-content/plugins/Pure-Highlightjs_1.0/assets/pure-highlight.css?ver=0.1.0 HTTP/1.1
$ Host:www.taohui.pub  # 然后按两次回车

HTTP/1.1 200 OK
Server: openresty/1.15.8.3
Date: Tue, 02 Jun 2020 13:38:25 GMT
Content-Type: text/css
Content-Length: 108
Last-Modified: Thu, 27 Dec 2018 07:35:33 GMT
Connection: keep-alive
ETag: "5c2480c5-6c"
Expires: Tue, 09 Jun 2020 13:38:25 GMT
Cache-Control: max-age=604800
Accept-Ranges: bytes

pre.pure-highlightjs {
    background-color: transparent;!important;
    border: none;
    padding: 0;
}
```

以上方式得到的 HTTP 响应是看不见不可见字符(如空格、换行)的，可以使用 Wireshark 抓包工具，通过查看十六进制信息来查看这些不可见字符。

> RFC规定，如果请求中没有携带Host头部，一律返回400 Bad Request，目前基本Web服务器都遵循这一规则。

# URI

URL（Uniform Resource Locator），表示资源的位置，期望提供查找资源的方法；

URN（Uniform Resource Name），期望为资源提供持久的、位置无关的标识方式，允许简单地将多个命名空间映射到单个 URN 命名空间；

URI（Uniform Resource Identifier），用以区分资源，是 URL 和 URN 的超集，用以取代 URL 和 URN 概念。

URI 的 ABNF 格式：URI = `scheme ":" hier-part [ "?" query ] [ "#" fragment ]`

- scheme = `ALPHA *( ALPHA / DIGIT / "+" / "-" / "." )`
  - 如：http, https, ftp,mailto,rtsp,file,telnet
- query = `*( pchar / "/" / "?" )`
- fragment = `*( pchar / "/" / "?" )`
- hier-part = `"//" authority path-abempty / path-absolute / path-rootless / path-empty`
  - userinfo = `*( unreserved / pct-encoded / sub-delims / ":" )`
  - host = `IP-literal / IPv4address / reg-name`
  - port = `*DIGIT`
- path = `path-abempty/ path-absolute/ path-noscheme / path-rootless / path-empty`
  - path-abempty = `*( "/" segment )`，以/开头的路径或者空路径
  - path-absolute = `"/" [ segment-nz *( "/" segment ) ]`，以/开头的路径，但不能以//开头
  - path-noscheme = `segment-nz-nc *( "/" segment )`，以非:号开头的路径
  - path-rootless = `segment-nz *( "/" segment )`，相对path-noscheme，增加允许以:号开头的路径
  - path-empty = `0<pchar>`，空路径

相对 URI，URI-reference = `URI/relative-ref`

- relative-ref = `relative-part [ "?" query ] [ "#" fragment ]`
  - relative-part = `"//" authority path-abempty / path-absolute / path-noscheme / path-empty`

![URI 组成](../../images/URI_example.jpg)

## URI 编码

Q：为什么需要对 URI 进行编码？

A：

1. 传递数据的时，可能会使用到那些用作分割符的保留字符，如在 baidu 中搜索 `?#!` 这样的内容，假设不做编码的话，请求会是 `https://www.baidu.com/s?wd=?#!`，而 `?` 是一个保留字符，本来想要搜索 `?#!`，结果其实只能搜索到 `#!`；
2. 需要对可能存在歧义的数据进行编码
   1. 不在 ASCII 码范围内的字符
   2. ASCII 码中不可显示的字符
   3. URI 中规定的保留字符
   4. 不安全字符(传输环节中可能会被不正确处理)，如空格、引号、尖括号等

URI 的编码方式为百分号编码：pct-encoded = `"%" HEXDIG HEXDIG`。

非 ASCII 码字符(例如中文):建议先 UTF8 编码，再 US-ASCII 编码；对 URI 合法字符，编码与不编码是等价的。

# HTTPS

HTTPS(安全套接字层超文本传输协议)：在HTTP的基础上加入了 SSL(Secure Sockets Layer) 协议，SSL依靠证书来验证服务器的身份，并为浏览器和服务器之间的通信加密。简单来说，HTTPS协议是由 SSL+HTTP 协议构建的可进行加密传输、身份认证的网络协议，要比HTTP协议安全。

HTTPS和HTTP的区别主要如下：
- HTTPS协议需要到 CA 申请证书，一般免费证书较少，因而需要一定费用。
- HTTP是超文本传输协议，信息是明文传输，HTTPS则是具有安全性的ssl加密传输协议。
- HTTP和HTTPS使用的是完全不同的连接方式，用的端口也不一样，前者是80，后者是443。
- HTTP的连接很简单，是无状态的；HTTPS协议是由SSL+HTTP协议构建的可进行加密传输、身份认证的网络协议，比HTTP协议安全。

# 代理

## 正向代理

一般情况下，如果没有特别说明，代理技术默认指正向代理(Forward Proxy)技术。

正向代理是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后代理向原始服务器转交请求并将获得的内容返回给客户端。**客户端才能使用正向代理，客户端必须设置正向代理服务器，当然，前提是知道正向代理服务器的IP地址、代理程序的端口**。

使用正向代理服务器的作用：

1. 访问原本无法访问的资源，如Google；
2. 做Cache，加速访问资源；
3. 对客户端访问授权，上网进行认证；
4. 代理可以记录用户访问记录(上网行为管理)，隐藏访问者行踪：由于代理服务器代替客户端来直接与服务器交互，所以服务器并不知道访问自己的实际用户是谁。

## 反向代理

反向代理(Reverse Proxy)对外是透明的，访问者并不知道自己访问的是一个代理。实际运行方式：以代理服务器来接受internet上的连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给internet上请求连接的客户端，此时代理服务器对外就表现为一个服务器。

**客户端只会得知反向代理的IP地址，而不知道在代理服务器后面的服务器簇的存在，也就是说，对于客户端而言，代理服务器就像是原始服务器，且客户端不需要进行任何特别的设置**。

使用反向代理服务器的作用：

1. 保护内网的安全，可以使用反向代理提供WAF功能，阻止web攻击
   - 大型网站通常将反向代理作为公网访问地址，Web服务器是内网；
2. 负载均衡，通过反向代理服务器来优化网站的负载。

## 区别

这里引用知乎上的一个图片：（详见知乎问题[反向代理为何叫反向代理](https://www.zhihu.com/question/24723688)）

![](https://pic4.zhimg.com/80/2582ac2a1366e3e12acf274265ea80f3_hd.jpg)

- 位置不同：
  - 正向代理中，代理和客户端在同一个LAN中，对服务端透明；
  - 反向代理中，代理和服务端在同一个LAN中，对客户端透明。
- 代理对象不同：
  - 正向代理：代理客户端，服务端不知道实际发起请求的客户端；
  - 反向代理：代理服务端，客户端不知道实际提供服务的服务端；
- 用途不同：
  - 正向代理为防火墙内的局域网客户端提供访问 internet 的途径；
  - 反向代理将防火墙后面的服务器提供给 internet 访问；
- 安全性不同：
  - 正向代理允许客户端通过它访问任意网站并且隐藏客户端自身，因此必须采取安全措施以确保仅为授权的客户端提供服务； 
  - 反向代理都对外都是透明的，访问者并不知道自己访问的是哪一个代理。

在正、反向代理中，代理服务器所做的都是代为收发请求和响应，但从结构上看正好左右互换了以下，所以后出现的代理方式就叫做反向代理。
