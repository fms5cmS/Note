# HTTP 报文

## 请求报文

当浏览器请求一个网页时，它会向网络服务器发送一系列不能被直接读取的信息，因为这些信息是作为 HTTP 信息头的一部分来传送的。 下面是一个请求报文的实例（以访问 B 站的某个视频为例）：

```http
GET /x/web-interface/card?mid=2722268&photo=true HTTP/1.1
Host: api.bilibili.com
Connection: keep-alive
Accept: application/json, text/plain, */*
Origin: https://t.bilibili.com
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/68.0.3440.106 Safari/537.36
Referer: https://t.bilibili.com/
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Cookie:........................
```

说明：起始行的`GET`表示请求访问服务的类型，称为方法（method）；随后的字符串指明了请求访问的资源对象，也叫做请求 URI（request-URI），最后的`HTTP/1.1`为 HTTP 的版本号，用来提示客户端使用的 HTTP 协议功能。

请求消息的结构：

- ==一个请求行（请求方法、请求 URI、HTTP 协议版本）；若干消息头即首部字段；以及实体内容==
  - 其中的一些消息头和实体内容都是可选的，==消息头和实体内容之间要用空行（CR+LF）隔开==。
    - CR（Carriage Return，回车符：16 进制 0x0d）
    - LF（Line Feed，换行符：16 进制 0x0a）

在上面的例子中，第二行以后均为请求首部字段，而没有实体内容。

### 请求方法

用于定义对于资源的操作。常用有 GET、POST 等。从定义上讲有各自的语义，但是具体如何实现，取决于自己的操作。

- `GET`：获取资源，大量的性能优化都针对该方法，幂等方法

  - `GET`方法用来请求访问已被 URI 识别的资源。指定的资源经服务器端解析后返回响应内容。也就是说，如果请求的资源是文本，那就保持原样返回；如果是像 CGI（Common Gateway Interface，通用网关接口）那样的程序，则返回经过执行后的输出结果。

  ```http
  GET /index.html HTTP/1.1
  Host: www.bilibili.com
  ```

  - 上面请求的响应会返回 index.html 的页面资源

- `POST`：常用于提交 HTML FORM 表单、新增资源等

  - `GET` 方法也可用来传输内容主题，但一般不用 `GET` 方法传输，而使用 `POST` 方法。

  ```http
  POST /submit.cgi HTTP/1.1
  Host: www.hackr.jp
  Content-Length: 1560（1560字节的数据）
  ```

  - 上面请求的响应会返回 submit.cgi 接收数据的处理结果

- `PUT`：更新资源，带条件时是幂等方法

  - `PUT`方法用来传输文件。就像 FTP 协议的文件上传一样，要求在请求报文的主体中包含文件内容，然后保存到请求 URI 指定的位置。
  - 由于 HTTP/1.1 的`PUT`方法自身不带验证机制，任何人都可以上传文件，存在安全问题，因此一般 Web 网站不用该方法。若是配合 Web 应用程序的验证机制，或架构设计采用 REST（REpresentational State Transfer，表征状态转移）标准的同类 Web 网站，就可能会开放使用 PUT 方法。

- `HEAD`：获得报文首部，服务器不发送响应体，幂等方法

  - `HEAD`方法和`GET`方法一样，只是不返回报文主体部分。用于确认 URI 的有效性及资源更新的日期时间等。

- `DELETE`：删除资源，幂等方法

  - `DELETE`方法用来删除文件，是与`PUT`相反的方法。`DELETE`方法按请求 URI 删除指定的资源。
  - 但是，HTTP/1.1 的`DELETE`方法本身和`PUT`方法一样不带验证机制，所以一般的 Web 网站也不使用`DELETE`方法。当配合 Web 应用程序的验证机制，或遵守 REST 标准时还是有可能会开放使用的。

- `OPTIONS`：显示服务器对访问资源支持的方法，幂等方法，主要用于跨域访问时

```shell
curl 网址 -X OPTIONS -I  # 查询该网站允许的方法
```

- `TRACE`：回显服务器收到的请求，用于定位问题。有安全风险，很少使用，Nginx 于 2007 年不再支持该方法

- `CONNECT`：建立 tunnel 隧道

  - `CONNECT`方法要求在与代理服务器通信时建立隧道，实现用隧道协议进行 TCP 通信。主要使用 SSL（Secure Sockets Layer，安全套接层）和 TLS（Transport Layer Security，传输层安全）协议把通信内容加 密后经网络隧道传输。

  - 该方法的格式：

    ```http
    CONNECT 代理服务器名:端口号 HTTP版本
    ```

### 请求头

这里列出部分常见请求头：

| 请求头          | 描述                                                         |
| --------------- | ------------------------------------------------------------ |
| Accept          | 指定浏览器或其他客户端可以处理的 MIME 类型。它的值通常为 **image/png** 或 **image/jpeg** |
| Accept-Charset  | 指定浏览器要使用的字符集。比如 ISO-8859-1                    |
| Accept-Encoding | 指定编码类型。它的值通常为 **gzip** 或**compress**           |
| Accept-Language | 指定客户端首选语言，servlet 会优先返回以当前语言构成的结果集，如果 servlet 支持这种语言的话。比如 en，en-us，ru 等等 |
| Cookie          | 返回先前发送给浏览器的 cookies 至服务器                      |
| Host            | 指出原始 URL 中的主机名和端口号                              |
| User-Agent      | 指明客户端的类型信息，服务器可以据此对资源的表述做抉择       |
| Connection      | 表明客户端是否可以处理 HTTP 持久连接。持久连接允许客户端或浏览器在一个请求中获取多个文件。**Keep-Alive** 表示启用持久连接 |
| Referer         | 浏览器对来自某一页面的请求自动添加的头部（服务器端常用于统计分析、缓存优化、防盗链等功能） |
| From            | 主要用于网络爬虫，告诉服务器如何通过邮件联系到爬虫的负责人   |

### 请求体

即实体内容，约定用户的表单数据向服务端传递格式。举例一：

```html
<form action="http://www.baidu.com/aaa" method="GET">
  <input type="text" name="username" />
  <input type="password" name="password" />
  <input type="submit" value="提交" />
</form>
```

上面的表单中，当录入数据后，点击提交，数据以下方式提交：

```http
GET /aaa?username=tom&password=123 HTTP/1.1
请求头
请求头
空行
```

举例二：

```html
<form action="http://www.baidu.com/aaa" method="POST">
  <input type="text" name="username" />
  <input type="password" name="password" />
  <input type="submit" value="提交" />
</form>
```

上面的表单中，当录入数据后，点击提交，数据以以下方式提交：

```http
GET /aaa HTTP/1.1
请求头
请求头

username=tom&password=123 请求体作用：存放客户端向服务端传递的数据
```

## 响应报文

当服务器接收到浏览器的请求后会做出响应给浏览器。下面是一个访问 B 站某个视频后得到的响应消息：

```http
HTTP/1.1 200 OK
Date: Sun, 12 Aug 2018 07:22:56 GMT
Content-Type: text/html; charset=utf-8
Transfer-Encoding: chunked
Connection: keep-alive
gear: 1
vikingrCache: 60000
Vikingr-Cache-TTL: 31581
Vary: Origin,Accept-Encoding
Content-Encoding: gzip
Expires: Sun, 12 Aug 2018 07:23:26 GMT
Cache-Control: max-age=30
X-Cache: HIT from cn-sdqd2-cmcc-w-02.hdslb.com
```

说明：在起始行开头的 `HTTP/1.1`表示服务器对应的 HTTP 版本；`200 OK`表示请求的处理结果的状态吗（status code）和原因短语（reason-phrase）；下一行显示了创建响应的日期时间，是首部字段（header field）内的一个属性。

响应消息的结构：

- ==一个状态行（HTTP 协议版本、状态码（表示请求成功或失败的数字代码）、用以解释状态码的原因短语）、若干消息头即首部字段、以及实体内容==
- 其中的一些消息头和实体内容都是可选的，==消息头和实体内容之间要用空行（CR+LF）隔开==。

上面的例子中从第二行开始均为首部字段，并没有实体内容。

### 状态码

状态码的职责是当客户端向服务器端发送请求时，描述请求的处理结果。借助状态码，用户可以知道服务器端是正常处理了请求，还是出现了错误。

状态码以三位数字和原因短语组成。数字的第一位指明了响应类别，后两位无分类。五种响应类别：

|      | 类别                             | 原因短语                                             |
| ---- | -------------------------------- | ---------------------------------------------------- |
| 1xx  | Informational（信息性状态码）    | 请求已接收到，需要进一步处理才能完成，HTTP1.0 不支持 |
| 2xx  | Success（成功状态码）            | 请求正常处理完毕                                     |
| 3xx  | Redirection（重定向状态码）      | 需要重定向，进行附加操作以完成请求                   |
| 4xx  | Client Error（客户端错误状态码） | 客户端出现错误                                       |
| 5xx  | Server Error（服务器错误状态码） | 服务器处理请求出错                                   |

- 2xx
  - 200 OK: 成功返回响应。
  - 201 Created: 有新资源在服务器端被成功创建。
  - 202 Accepted: 服务器接收并开始处理请求，但请求未处理完成。这样一个模糊的概念是有意如此设计，可以覆盖更多的场景。例如异步、需要长时间处理的任务。
  - 204 No Content：成功执行了请求且不携带响应包体，并暗示客户端无需更新当前的页面视图。
  - 205 Reset Content：成功执行了请求且不携带响应包体，同时指明客户端需要更新当前页面视图。
  - 206 Partial Content：使用 range 协议时返回部分响应内容时的响应码。（多线程断点下载时会使用）。
- 3xx
  -  301 Moved Permanently：资源永久性的重定向到另一个 URI 中。
  - 302 Found：资源临时的重定向到另一个 URI 中。
  - 303 See Other：重定向到其他资源，常用于 POST/PUT 等方法的响应中。
  - 307 Temporary Redirect：类似302，但明确重定向后请求方法必须与原请求方法相同，不得改变。
  - 308 Permanent Redirect：类似301，但明确重定向后请求方法必须与原请求方法相同，不得改变。
- 4xx
  -  400 Bad Request：服务器认为客户端出现了错误，但不能明确判断为以下哪种错误时使用此错误码。如HTTP请求格式错误。
  -  401 Unauthorized：用户认证信息缺失或者不正确，导致服务器无法处理请求。
  - 403 Forbidden：服务器理解请求的含义，但没有权限执行此请求。
  -  410 Gone：服务器没有找到对应的资源，且明确的知道该位置永久性找不到该资源



### 响应头

这里列出部分响应头。

| 响应头           | 描述                                                         |
| ---------------- | ------------------------------------------------------------ |
| Allow            | 指定服务器支持的 request 方法（GET，POST 等等）              |
| Connection       | 命令浏览器是否要使用持久的 HTTP 连接。**close**意味着浏览器不使用持久 HTTP 连接，而 keep-alive 意味着使用持久化连接。 |
| Content-Encoding | 指定传输时页面的编码规则                                     |
| Content-Language | 表述文档所使用的语言，比如 en， en-us,，ru 等等              |
| Set-Cookie       | 指明当前页面对应的 cookie                                    |
| Date             | 响应时间                                                     |
| Server           | 指明服务器上所用软件的信息，用于帮助客户端定位问题或者统计数据 |
| Accept-Ranges    | 告诉客户端服务器上该资源是否允许 range 请求                  |

响应头 Cache-Control：指定响应文档能够被安全缓存的情况。通常取值为 **public**，**private**或**no-cache** 等等。

- Public：在 HTTP 请求返回过程中，返回内容所经过的路径中(包括中间的 HTTP 代理服务器、发出该请求的客户端浏览器)都可以进行对返回内容的缓存操作；
- Private：只有发出该请求的客户端浏览器才可以进行缓存操作且只能使用私有缓存；
- No-cache：任何一个节点都不能进行缓存操作。

`max-age=秒数`：指定缓存的过期时间。

`s-maxage=秒数`：在代理服务器中该属性会代替上面的属性来设置缓存的过期时间。

`must-revalidate`：缓存过期后，必须重新发送请求获取这一部分数据，再来验证这一部分内容是否真的过期，而不能直接使用本地缓存。

`proxy-revalidate`：和上面的一样，不过是用在缓存服务器中的

# HTTP 连接

http 连接的常见流程：

1. 浏览器根据域名机械出主机名
2. 浏览器查询这个主机名的 IP 地址（DNS）
3. 浏览器获得端口号（80）
4. 浏览器发起到该 IP 地址 80 端口的连接
5. 浏览器向服务器发送一条 HTTP GET 报文
6. 浏览器从服务器读取 HTTP 响应报文
7. 浏览器关闭连接

Connection 头部：

- Keep-Alive 长连接
  - 客户端请求长连接 Connection: Keep Alive
  - 服务器表示支持长连接 Connection: Keep Alive
  - 客户端复用连接
  - HTTP/1.1 默认支持长连接，所以传递 Connection: Keep Alive 无意义
- Close 短连接
- 对代理服务器的要求：不转发 Connection 列出头部，该头部仅与当前连接相关

注意：Connection 仅针对**当前**连接有效

错误使用长连接：假设 client 向古老的正向 proxy 发送请求(携带 Connection:KeepAlive)，而 proxy 不支持长连接，无法识别 Connection 头部，就会将其原样转发给 server，而 server 是支持长连接的，收到 proxy 的 Connection 头部，以为要建立长连接，就会响应给 proxy 一个 Connection 头部，而 proxy 无法识别又将其转发给 client，最后 client、server都以为自己建立的长连接，实际上其实都是短连接。那么 client 在复用自己以为的长连接时就会出现无法响应的情况。

为防止以上错误，故增加了 Proxy-Connection 头部：旧的代理服务器不识别该头部，会退化为短连接，而新的代理服务器可以识别，会与客户端建立长连接，于服务器交互时会用 Connection 替代 Proxy-Connection 头部。



#  会话追踪

==HTTP 协议是一种无状态的协议==，WEB 服务器本身不能识别出哪些请求是同一个浏览器发出的 ，浏览器的每一次请求都是完全孤立的，即使 HTTP1.1 支持持续连接，但当用户有一段时间没有提交请求，连接也会关闭。

==作为 web 服务器，必须能够采用一种机制来唯一地标识一个用户，同时记录该用户的状态。==

- 会话和会话状态：

  - Web 应用中的会话是指一个客户端浏览器与 Web 服务器之间连续发生的一系列请求和响应过程。

  - Web 应用的会话状态是指 WEB 服务器与浏览器在会话过程中产生的状态信息，==借助会话状态，WEB 服务器能够把属于同一会话中的一系列的请求和响应过程关联起来。==

    为了让 Web 服务器从大量请求消息中识别出来自同一个浏览器的访问请求（即这些请求消息属于同一个会话），需要浏览器对其发出的每个请求消息进行标识：同一个会话中的请求消息都附带同样的标识号，这个标识号就称之为==会话 ID（SessionID）==。

- 在 Servlet 规范中，用以下两种机制完成会话追踪

  - Cookie
  - Session

## Cookie

Cookie 机制采用的是在==客户端保存 HTTP 状态信息==的方案。

Cookie 是在浏览器访问 Web 服务器的某个资源时，==由 WEB 服务器在 HTTP 响应消息头中附带传送给浏览器的一个小文本文件。==一旦 WEB 浏览器保存了某个 Cookie，那么它在以后每次访问该 WEB 服务器时，==都会在 HTTP 请求头中将这个 Cookie 回传给 WEB 服务器。==

底层的实现原理： Web 服务器通过在 HTTP 响应消息中增加 **Set-Cookie 响应头字段**将 Cookie 信息发送给浏览器，浏览器则通过在 HTTP 请求消息中增加 **Cookie 请求头字段**将 Cookie 回传给 Web 服务器。

- 一个 Cookie 只能标识一种信息，它至少含有一个标识该信息的名称（NAME）和设置值（VALUE）。
- Cookie 是键值对，可以设置多个。

Cookie 的属性：

- max-age 和 expires 设置过期时间
  - 会话 Cookie 与持久化 Cookie 的区别：
    - 如果不设置过期时间，则表示这个 Cookie 生命周期为浏览器会话期间，只要关闭浏览器窗口，Cookie 就消失了。这种生命期为浏览器会话期的 Cookie 被称为会话 Cookie。==会话 Cookie 一般不保存在硬盘上而是保存在内存里==。
    - 如果设置了过期时间，浏览器就会把 Cookie 保存到硬盘上，关闭后再次打开浏览器，这些 Cookie 依然有效直到超过设定的过期时间。
- domain：可以访问该 Cookie 的域名
- path：访问该 Cookie 的页面路径。如 domain=abc.com，path=/test，则只用 /test 路径下的页面可以访问
  - Cookie 的作用范围：可以作用于当前目录和当前目录的子目录，不能作用于当前目录的上一级目录
- Secure 代表该 Cookie 只在 https 请求时才会带上
- 设置 HttpOnly 后无法通过 JS 的 document.cookie 访问该 Cookie 的内容

## Session

session 机制采用的是==在服务器端保存 HTTP 状态信息==的方案。

当程序需要为某个客户端的请求创建一个 session 时，服务器首先检查这个==客户端的请求==里是否包含了一个 session 标识(即 SessionId)；

如果已经包含一个 SessionId 则说明以前已经为此客户创建过 session，服务器就按照 Sessionid 把这个 session 检索出来使用(如果检索不到，可能会新建一个，这种情况可能出现在服务端已经删除了该用户对应的 session 对象，但用户人为地在请求的 URL 后面附加上一个 JSESSIONID 的参数)。

如果客户请求不包含 SessionId，则为此客户创建一个 session 并且生成一个与此 session 相关联的 SessionId，==这个 sessionid 将在本次响应中以 Cookie 的形式返回给客户端保存==。

- 客户端发出请求——>服务端创建 HttpSession 对象——>响应（Set-Cookie）以 Cookie 形式向客户端传回 JSESSIONID——>客户端下次发出请求时，请求中（Cookie）会携带 JSESSIONID 参数

保存 SessionId 的几种方式：

- ==Session cookie 方式==：采用 Cookie，这样在交互的过程中浏览器可以自动的按照规则把这个标识发送给服务器；
  - ==默认方式==。系统会创建一个名为 ==JSESSIONID== 的输出 Cookie，这称之为 session cookie,以区别 persistent cookies(也就是我们通常所说的 cookie),session cookie 是==存储于浏览器内存中的，并不是写到硬盘上的==，通常看不到 JSESSIONID，但是当把浏览器的 cookie 禁止后，Web 服务器会采用 URL 重写的方式传递 Sessionid，这时地址栏看到。
  - Session cookie 针对某一次会话而言，会话结束后也会随之消失。而 persistent cookie 是存在于客户端硬盘上的一段文本。
- 由于 Cookie 可以被人为的禁用，必须有其他的机制以便在 Cookie 被禁用时仍能把 SessionId 传给服务器。经常采用 **URL 重写方式**，就是把 SessionId 附加在 URL 路径后面。附加的方式也有两种，一种是作为 URL 路径的附加信息，另一种是作为查询字符串附加在 URL 后面。

## 对比

Session 的实现依赖于 Cookie，Session 的唯一标识 sessionId 存放在客户端。

区别：

- Cookie 数据保存在客户端，Session 数据保存在服务器端；
- Cookie 相对于 Session 不是很安全；
- Session 会在一定时间内保存在服务器上。当访问增多，会比较占用服务器性能，考虑到减轻服务器性能，应使用 Cookie；
- 单个 Cookie 保存的数据不超过 4K
- 使用场景：
  - 将登录信息等重要信息存放为 Session；
  - 其他信息如果需要保留，可放在 Cookie 中。
  - 购物车的实现最好使用 Cookie，但 Cookie 可以在客户端禁用，这时，使用 Cookie+数据库实现，当从 Cookie 不能取出数据时，从数据库获取

服务器要保存所有人的 session，如果访问过多，这对于服务器就是一个巨大的开销，严重限制了服务器扩展能力，为了防止用户多次请求发送到不同服务器上，需要将 session 在服务器之间搬来搬去。而如果将所有的 session 集中存储在一个地方，所有的服务器访问这里的数据，虽然不需要复制 session id，但增加了单点失败的可能性，存储 session 的服务器挂了，所有人都需要重新登录。

如果 session 由每个客户端保存的话就会减轻负担，但是不保存这些 session 的话，如何验证客户端发送过来的 session 确实是服务端生成的呢？所以关键在于验证，验证其是否为合法登录的用户。

# 验证-token

[彻底理解 Cookie、Session、Token](https://www.cnblogs.com/moyand/p/9047978.html)

token 是用户身份的验证方式，通常叫它：令牌。应用场景：无状态、可扩展；支持移动设备；跨程序调用；安全

用户登录：

1. 用户首次登录成功(注册也是一种可以适用的场景)后, 服务端就会生成一个 token 值(一般会包含 uid)，会在服务器保存 token 值(保存在数据库中)，再将这个 token 值返回给客户端；
2. 客户端拿到 token 值之后，进行本地保存(如保存在 cookie 中)；
3. 当客户端再次发送任何网络请求(一般不是登录请求)的时候，都会将这个 token 值附带到参数中发送给服务器；
4. 服务器接收到客户端的请求之后,会取出 token 值与保存在本地(数据库)中的 token 值做对比

- 如果两个 token 值相同， 说明用户登录成功过!当前用户处于登录状态!
- 如果没有这个 token 值, 则说明没有登录成功
- 如果 token 值不同: 说明原来的登录信息已经失效,让用户重新登录

为了防止别人伪造 token，需要对数据做一个签名，如使用 HMAC-SHA256 等摘要算法，加上一个只有服务端知道的秘钥，对用户 uid 进行签名，还可以加上时间信息，控制 token 的有效期限，避免 token 泄露长期有效。由于秘钥只有服务端知道，别人没法伪造，只有服务端能够解析该 token。

最简单的 token 组成：uid(用户唯一的身份标识)、time(当前时间的时间戳)、sign(签名，由 token 的前几位+盐以哈希算法压缩成一定长的十六进制字符串，可以防止恶意第三方拼接 token 请求服务器)。还可以把不变的参数也放进 token，避免多次查库。如：

```
# SECRET 是密钥
expireTime + md5(expireTime+uid+SECRET)+"uid"+uid
```

session 和 token 并不矛盾，作为身份认证 token 安全性比 session 好，因为每个请求都有签名还能防止监听以及重放攻击，而 session 就必须靠链路层来保障通讯安全了。

优势：

- 无状态、可扩展：在客户端存的 token 是无状态的，且能被扩展。基于这种无状态和不存储 session 信息，负载均衡服务器能将用户信息从一个服务传到其他服务器上
- 安全性：请求中发送 token 而不再是发送 cookie，能阻止 CSRF(跨站请求伪造)。
  - 即使在客户端使用 cookie 存储 token，cookie 也仅仅是一个存储机制而不是用于认证，不将信息存储在 session 中，可以不用再对 session 操作。
- 服务端对 token 进行校验时，不需要额外的数据，也就是没有像 session id 那样的存储负担
- 通过 token 可以方便解析出过期时间，防止 token 永久有效
- 可以方便解析出用户标识，做用户校验

具体实现方案：JWT(JSON Web Token)

## JWT

[JSON Web Token](https://jwt.io/introduction/) 是一种用以产生访问令牌的开源标准，是目前最流行的跨域认证解决方案。一般用于处理用户身份验证或数据信息交换，是目前最流行的跨域认证解决方案！！！！

如，服务器可以生成一个令牌，该令牌具有“以管理员身份登录”的声明，并将其提供给客户端。然后，客户端可以使用该令牌来证明它以 admin 身份登录。令牌是由一方的私钥（通常是服务器的私钥）签名的，因此，双方（另一方已经通过某种适当且可信赖的方式已经拥有相应的公钥）能够验证令牌是否合法。

特点：

- 紧凑(数据小，可通过 URL、POST 参数、请求头发送，且传输速度快)
- 自包含(使用 payload 数据块基于用户必要且不隐私的数据，可有效减少数据库访问次数，提高代码性能)

JWT 依赖于其他基于 JSON 的标准：JSON Web 签名和 JSON Web 加密。

单点登录是广泛使用 JWT 的一项功能，因为它开销很小，且能轻松跨域使用。

JWT 也是一种非常方便地多方传递数据的载体，因为其可以使用数据加密来保证数据的有效性、安全性。

## JWT 原理

服务器认证以后，生成一个 JSON 对象，发回给用户，以后，用户与服务端通信的时候，都要发回这个 JSON 对象。服务器完全只靠这个对象认定用户身份。为了防止用户篡改数据，服务器在生成这个对象的时候，会加上签名。

服务器就不保存任何 session 数据了，也就是说，服务器变成无状态了，从而比较容易实现扩展。

## JWT 数据结构

JWT 的数据结构是：`Header.Payload.Signature`：(使用`.`来分隔)

### Header(头部)

描述 JWT 的元数据，通常包含：

- alg：表示签名的算法（algorithm），如 HMAC SHA256 或者 RSA 等，默认是 HMAC SHA256（写成 HS256）
- typ：表示这个令牌（token）的类型（type），JWT 令牌统一写为`JWT`

如：

```json
{
  "alg": "HS256",
  "typ": "JWT"
}
```

最后使用 base64UrlEncode 算法对上面的 JSON 对象转换，使其成为 JWT 的一部份。

### Payload(负载)

存放实际需要传递的数据(用户必要且不隐私的)，分为三个部分：

- 1.已注册信息(Registered claims)，JWT 规定了 7 个官方字段，供选用(不强制但推荐)：
  - iss (issuer 签发人)、exp (expiration time 过期时间)、sub (subject 主题)、aud (audience 受众，接受 JWT 的一方)、nbf (Not Before 生效时间)、iat (Issued At 签发时间)、jti (JWT ID 编号)：
- 2.公开数据(Public claims)，需要额外注册。
- 3.私有数据(Private claims)，用于在同意使用它们的各方之间共享信息，并且不是注册的或公开的声明，可以随意定义。

如：

```json
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

同样会使用 base64UrlEncode 算法对该 JSON 对象转换，使其成为 JWT Token 的一部份，注意 base64UrlEncode 算法是可逆的，所以，**不要在 JWT 的 payload 中放置敏感信息，除非它们是加密的**。

### Signature(签名)

这是由开发者提供的信息，是服务器验证传递的数据是否安全有效的标准。**在生成 JWT 最终数据之前，先使用 base64UrlEncode 将 header 和 payload 加密，并使用`.`连接，然后再使用 header 中定义的加密算法对加密后的数据和签名信息进行加密，得到 Signature：**

```json
HMACSHA256( 
  base64UrlEncode(header) + "." + base64UrlEncode(payload), 
  secret)
```

Signature 是由 Header、Payload 和 secret 的算法组成的，可以用来校验消息是否被篡改。

## 执行流程

客户端收到服务器返回的 token，可以储存在 Cookie 里面，也可以储存在 localStorage。更好的做法是放在 HTTP 请求的头信息`Authorization`字段里面。

```http
Authorization: Bearer <token>
```

![](https://auth0.com/learn/wp-content/uploads/2016/01/17.png)

也可以，跨域的时候，JWT 就放在 POST 请求的数据体里面。

## 使用

[入门教程](http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)

[Java 中的使用](../../Java/2-Web/JWT.md)

## 注意

使用 JWT 实现 SSO 时，要注意 token 时效性，token 时保存在客户端的令牌数据，如果永久有效，则有被劫持的可能。token 在设计时，可以考虑一次性有效或一段时间内有效。如果设置有效市场，则需要考虑是否需要刷新 token 有效期问题。

# 跨域问题

## 同源策略

一级域名(顶级域名)：只有一个`.`，且`.`的左边要有字段，如，`zzk.com`；

二级域名：如，`www.zzk.com`、`news.zzk.com`；

浏览器的**同源策略(Same-origin Policy)**：在一个浏览器中访问的网站不能访问另一个网站中的数据，除非这两个网站具有相同的 Origin，也即是拥有**完全相同的协议、主机地址以及端口**。

但是，一个大型网站通常由一系列子域，这些子域之间交换数据会受到同源策略的限制，为了绕开该限制，业界提供了一些方法，如：利用 Cookie 更改`document.domain`属性，跨文档消息，JSONP 以及 CORS 等。

## Cookie 解决跨域

- 更改 Cookie 的`domain`属性(该属性代表可以访问此 Cookie 的域名)：解决二级域名不同的跨域

  1. 在 Cookie 中设置`domain=一级域名`；

  2. 将该 Cookie 放在一级域名下，同时设置 Cookie 的 Path 属性为根目录`/`

     - 设置在根目录，根目录下的所有目录下的页面和代码都能获取到 Cookie
     - 如果设置为`test`，则只有 test 下的目录中的页面和代码才能获取 Cookie

  3. 这样，该一级域名下的二级域名也可以读到该 Cookie，于是同一一级域名下的不同二级域名间的数据也可以互相访问了

     ```
     domain=fms5cms.com
     A.fms5cms.com              cookie:  domain=A.fms5cms.com   path="/"
     B.fms5cms.com              cookie:  domain=B.fms5cms.com   path="/"
     A.fms5cms.com/test/cc       cookie:  domain=A.fms5cms.com   path="/test/cc"
     A.fms5cms.com/test          cookie:  domain=A.fms5cms.com   path="/test"
     ```

## JSONP 解决跨域

`<link>`、`<img>`、`<script>`等标签上的路径加载内容时，浏览器是允许跨域的

JSONP 是通过在文档中嵌入一个`<script>`标记来从另一个域中返回数据。如在页面中添加一个如下的标记：

```html
<script src="http://127.0.0.8887"></script>
```

当 8888 端口的服务读取该页面，该标记会向`http://127.0.0.8887`(8887 端口的服务)发送一个 GET 请求，此时也可以绕过同源策略。但是，仅支持 GET 请求。

## CORS 解决跨域！

CORS(Cross-Origin Resource Sharing)跨来源资源共享。

- 使用方式：设置响应头`Access-Control-Allow-Origin=允许访问的域(如：http://baidu.com)`，只能写入一个域
  - 该响应头用来记录可以访问该资源的域。在接收到服务端响应后，浏览器将会查看响应中是否包含`Access-Control-Allow-Origin`响应头。如果该响应头存在，那么浏览器会分析该响应头中所标示的内容。如果其包含了当前页面所在的域，那么浏览器就将知道这是一个被允许的跨域访问，从而不再根据 Same-origin Policy 来限制用户对该数据的访问。
  - 如果值为`*`则表示允许任何域来请求资源。

* prefligt 请求：

preflight 请求：发生 CORS 请求时，浏览器检测到跨域请求，会自动发出一个 OPTIONS 请求来检测本次请求是否被服务器接受。

一个 OPTIONS 请求一般会携带下面两个与 CORS 相关的头：

- `Access-Control-Request-Method` : 本次预检请求的请求方法。
- `Access-Control-Request-Headers`：本次请求所携带的自定义首部字段。这些字段是导致产生 OPTIONS 请求的一个原因。

服务端收到该预检请求后，会返回与 CORS 相关的响应头。主要会包括下面几个(可能还会有其他的有关 CORS 字段)：

- `Access-Control-Allow-Origin`: 服务器允许的跨域请求源
- `Access-Control-Allow-Methods`: 服务器允许的请求方法
- `Access-Control-Allow-Headers` : 服务器允许的自定义的请求首部字段

服务器通过 CORS 跨域请求后，下览器就会发生正式的数据请求。整个请求过程其实是发生了两次请求：

- 预检请求(预检请求只是一个检查的过程，它不会携带任何请求的参数)；
- 预检请求通过后的实际数据请求(该求才会真正的携带请求参数与服务器进行数据通信)。

[参考](https://segmentfault.com/p/1210000012470287/read)

---

- 简单请求

预检请求并不是 CORS 请求必须的请求过程，在一定条件下并不需要发生预检请求。如果该请求是一个简单请求的化，就不会发生预检请求。

- 满足以下所有条件的请求就是简单请求：
  - 请求方法方法必须是 GET、HEAD、POST 之一；
  - Content-Type：`text/plain`、`multipart/form-data`、`application/x-www-form-urlencoded`
  - 请求头限制，可以使用的有：Accept、Accept-Language、Content-Language、Content-Type、DPR、Downlink、Save-Data、Viewport-Width、Width
  - XMLHttpRequestUpload 对象均没有注册任何事件监听器
  - 请求中没有使用 ReadableStream 对象

实际项目中我们的请求格式可能是`application/json`格式编码，或者使用自定义请求头都会触发 CORS 的预检请求。
