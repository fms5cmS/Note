Web 使用一种名为 HTTP（HyperText Transfer Protocol，超文本传输协议）的协议作为规范，完成从客户端到服务器端等一系列运作流程，从而完成通信。其三项 WWW（World Wide Web）构建技术：

1. SGML（StandardGeneralized Markup Language，标准通用标记语言）作为页面的文本标记语言的 HTML（HyperText Markup Language，超文本标记语言）；
2. 作为文档传递协议的 HTTP ；
3. 指定文档所在地址的 URL（Uniform Resource Locator，统一资源定位符）。



# HTTP协议发展

注意：在同一个 TCP 连接中可以发送多个 HTTP 请求！

HTTP/0.9：只有一个 GET 命令，没有 HEADER 等描述数据的信息，服务器发送完毕就关闭 TCP 连接；

HTTP/1.0：增加了很多命令，增加 status code 和 header，多字符集支持、多部分发送、权限、缓存等；

HTTP/1.1：支持持久连接，增加 pipeline，增加 host 和其他一些命令；（当前使用的主要版本）

HTTP/2：所有数据以二进制传输，同一个连接里面发送多个请求不再需要按照顺序来，头信息压缩及推送等提高效率的功能



# 长连接

HTTP协议的初始版本中，每进行一次HTTP通信就要断开一次TCP连接。随着HTTP的普及，文档中可能包含有大量图片的情况增多，如果继续这样的话，每次的请求都会造成无谓的 TCP 连接建立和断开，增加通信量的开销。为了解决这一TCP连接问题，提出了持久连接（HTTP Persistent Connections，也称为 HTTP keep-alive 或HTTP connection reuse）的方法。其特点：只要任意一端没有明确提出断开连接，则保持TCP连接。

持久连接的好处在于减少了 TCP 连接的重复建立和断开所造成的额外开销，减轻了服务器端的负载。另外，减少开销的那部分时间，使HTTP 请求和响应能够更早地结束，这样 Web 页面的显示速度也就相应提高了。

持久连接使得多数请求以管线化（pipelining）方式发送称为可能。以前必须等待并收到前一次请求的响应后才能发送下一个请求，而管线化技术出现后，不必等待响应也可直接发送下一个请求。这样就能够做到同时并行发送多个请求，而不需要一个接一个地等待响应了。

- 响应头 Connection：告诉浏览器是否要使用持久的HTTP连接。
  - close 意味着浏览器不使用持久HTTP连接
  - keep-alive 意味着使用持久化连接。



# URI、URL、URN

URI（Uniform Resource Identifier）统一资源标识符，用字符串标识某一互联网资源。包含URL、URN。

URL（Uniform Resource Locator）统一资源定位符，表示资源在互联网上所处的位置。可见URL是URI的子集。

- `http://user:pass@host.com:80/path?query=string#hash`
  - `http`：通信协议，如HTTP、FTP协议
  - `user:pass@`：如果要访问的资源需要有特定身份，从这里携带用户信息，现在的 Web 开发几乎不用，因为有更好的方式来做用户认证
  - `host.com`：用于定位资源所在的服务器在互联网中的位置，可以是 IP 也可以是域名(会通过 DNS 协议解析为 IP)
  - `80`：端口。不同的 Web 服务可以监听不同端口，使用端口可以定位 host 找到的服务器上存放的许多 Web 服务中的某一个 Web 服务。默认 HTTP 协议使用80端口
  - `/path`：路由。定位 Web 服务中具体的某个内容
  - `query=string`：请求携带的参数
  - `hash`：锚点。如果找到的Web服务的某个内容文档很大，使用锚点可以直接找到文档中的某一片段

URN（Uniform Resource Name）统一资源命名，通过名字来表示资源，在资源移动之后还能被找到，目前还没有非常成熟的使用方案。



# HTTP客户端

- 最常用的HTTP客户端是浏览器。
- 也可使用 Git Bash 来做HTTP客户端
  - 输入 `curl baidu.com` 会发送HTTP请求并得到一个返回内容很少的响应
  - 输入 `curl www.baidu.com` 会得到响应的实体内容
  - 输入 `curl -v www.baidu.com` 详细展示请求、响应的 header 等很多信息，在响应实体内容前面有 curl 所展示一些响应实体内容相关的信息