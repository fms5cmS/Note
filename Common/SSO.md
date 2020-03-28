单点登录(Single Sign On)：在多个应用系统中，用户只需要登录一次就可以访问所有相互信任的应用系统。

三方登录：某系统使用其他系统的用户，实现本系统登录的方式。解决信息孤岛和用户不对等的实现方案。

# 同域下的SSO

- 两个问题及解决方法：
  - 跨域问题：
    - 将 Cookie 的 domain 属性设置为一级域名；
  - 不同应用的session是保存在自己的应用内，不共享的。
    - 通过将 session 持久化到 Redis 上，从而实现 session 共享。



## 手动实现SSO

使用Redis + Cookie + Jackson + Filter 实现单点登录。

- 单点登录过程说明：
  1. 客户端发出登录请求，服务端接收请求，Controller层调用Service进行实际的登录操作；
  2. 服务端产生Cookie，Cookie 的 name 自定义，value 为 sessionId，同时完成 domain 属性的更改；
  3. 把用户登录的 session 持久化到 Redis
     - key=sessionId(即Cookie的value)、value=User对象的Json数据，同时设置该数据的有效期；
  4. 服务端在响应中携带 Cookie 发送给客户端，客户端会自己进行保存；
  5. 客户端再次进行其他操作时，请求会携带Cookie；
  6. 服务端根据 Cookie 的 value 从 Redis 中查询用户；
  7. 如果用户存在，则证明已登录，然后即可进行此次的相关操作了。
- 用户登出：
  1. 从客户端的请求中读取 Cookie 的 value；
  2. 删除 Cookie；
  3. 根据 Cookie 的 value 从 Redis 中删除对应的数据；
  4. 服务端响应客户端：已退出。
- 时间重置问题：
  - 场景说明：假设设置的Cookie有效期为半小时(这里的Session是以Cookie实现的，所以Session的有效期即为Cookie的有效期)，假设购物狂登录后，一直在浏览相关网页，过了半个小时后，即使该用户还在浏览，Session也会失效，强制用户登录。
  - 为了解决该问题，浏览过程中每次访问页面都要重置Cookie的有效期！使用过滤器实现：过滤所有.do结尾的请求，在里面进行判断并重置Session的时间。



## SpringSession实现SSO

见[SpringSession笔记](../Spring/SpringSession.md)。



# 不同域下的SSO

同域下的单点登录是巧用了 Cookie 顶域的特性，如果是不同域呢？不同域之间 Cookie 是不共享的，怎么办？



## CAS 项目实现SSO

CAS 是 Yale 大学发起的一个开源项目，旨在为 Web 应用系统提供一种可靠的单点登录方法。有以下特点：

- 开源的企业级单点登录解决方案；
- CAS Server 为需要独立部署的 Web 应用；
- CAS Client 支持非常多的客户端(这里指单点登录系统中的各个 Web 应用)，包括 Java, .Net, PHP, Perl, Apache, uPortal, Ruby 等

从结构上看，CAS 包含两个部分：

- CAS Server ：需要独立部署，主要负责对用户的认证工作；
- CAS Client ：负责处理对客户端受保护资源的访问请求，需要登录时，重定向到 CAS Server。

CAS 实现的单点登录访问流程主要有以下步骤：

1. 访问服务：SSO客户端发送请求访问应用系统提供的服务资源；
2. 定向认证：SSO客户端会重定向用户请求到SSO服务器；
3. 用户认证：用户身份认证；
4. 发放票据：SSO服务器会产生一个随机的Service Ticket；
5. 验证票据：SSO服务器验证票据Service Ticket的合法性，验证通过后，允许客户端访问服务；
6. 传输用户信息：SSO服务器验证票据通过后，传输用户认证结果信息给客户端。

Q：为什么不是直接在SSO登录认证通过后，通过回调地址将用户信息返回给原业务系统，原业务系统直接设置登录状态？而是先通过票据验证再返回用户信息？

A：从安全性考虑，如果在SSO没有登录，而是直接在浏览器中敲入回调的地址，并带上伪造的用户信息，是不是业务系统也认为登录了呢？这是很可怕的。



### 流程

[参照](https://yq.aliyun.com/articles/636281)

这是 [CAS 官网](https://apereo.github.io/cas/6.0.x/index.html)上的标准流程图：

![CAS Web flow diagram](https://apereo.github.io/cas/6.0.x/images/cas_flow_diagram.png)

具体流程如下：

1. 用户访问app系统，app系统是需要登录的，但用户现在没有登录。
2. 跳转到CAS server，即SSO登录系统，SSO系统也没有登录，弹出用户登录页。
3. 用户填写用户名、密码，SSO系统进行认证后，将登录状态写入SSO的session，浏览器（Browser）中写入SSO域下的Cookie。
4. SSO系统登录完成后会生成一个ST（Service Ticket），然后跳转到app系统，同时将ST作为参数传递给app系统。
5. app系统拿到ST后，从后台向SSO发送请求，验证ST是否有效。
6. 验证通过后，app系统将登录状态写入session并设置app域下的Cookie。

此时跨域单点登录就完成了，以后再访问app系统时，app就是登录的，而访问app2系统的流程：

1. 用户访问app2系统，app2系统没有登录，跳转到SSO。
2. 由于SSO已经登录了，不需要重新登录认证。
3. SSO生成ST，浏览器跳转到app2系统，并将ST作为参数传递给app2。
4. app2拿到ST，后台访问SSO，验证ST是否有效。
5. 验证成功后，app2将登录状态写入session，并在app2域下写入Cookie。

app2系统不需要走登录流程，就已经是登录了。SSO，app和app2在不同的域，它们之间的session不共享也是没问题的。