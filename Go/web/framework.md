部分内容来自 [Go语言高级编程](https://books.studygolang.com/advanced-go-programming-book/)部分。

默认的`net/http`包中的`mux`不支持带参数的路由，所以只要路由带有参数，且这个项目的API数目超过了10，就尽量不要使用`net/http`中默认的路由。在Go开源界应用最广泛的router 是 [httpRouter](<https://github.com/julienschmidt/httprouter>)，很多开源的 router 框架都是基于 httpRouter 进行一定程度的改造的成果。

# 路由

httprouter 中使用的是显式匹配，所以在设计路由的时候需要规避一些会导致路由冲突的情况，如：

```
conflicts：
GET /user/info/:name
GET /user/:id

no conflict:
GET /user/info/:name
POST /user/:id
```

如果两个路由拥有一致的 http 方法和请求路径前缀，且在某个位置出现了 A 路由是 wildcard（指:id这种形式）参数，B 路由则是普通字符串，那么就会发生路由冲突。路由冲突会在初始化阶段直接panic：

```
panic: wildcard route ':id' conflicts with existing children in path '/user/:id'
```

因为 httprouter 考虑到字典树的深度，在初始化时会对参数的数量进行限制，所以在路由中的参数数目不能超过255，否则会导致 httprouter 无法识别后续的参数

除支持路径中的 wildcard 参数之外，httprouter 还可以支持 `*` 号来进行通配，不过 `*` 号开头的参数只能放在路由的结尾！


## 数据结构

httprouter 以及很多衍生框架(如 Gin) 存储路由的数据结构是压缩字典树(Radix Tree)，类似于字典树(Trie Tree)。

[字典树](../../Base/06-DataStructure&Algorithm/12A-StringMatch.md#Trie树)常用来进行字符串检索，例如用给定的字符串序列建立字典树。对于目标字符串，只要从根节点开始深度优先搜索，即可判断出该字符串是否曾经出现过，时间复杂度为 $O(n)$，n可 以认为是目标字符串的长度。

普通的字典树有一个比较明显的缺点，就是每个字母都需要建立一个子节点，这样会导致字典树的层数比较深，压缩字典树相对好地平衡了字典树的优点和缺点。

压缩字典树的每个节点上不只存储一个字母了：

![](https://books.studygolang.com/advanced-go-programming-book/images/ch6-02-radix.png)

使用压缩字典树可以减少树的层数，同时因为每个节点上数据存储也比通常的字典树要多，所以程序的局部性较好（一个节点的 path 加载到 cache 即可进行多个字符的对比），从而对CPU缓存友好。

路由树创建过程见：https://books.studygolang.com/advanced-go-programming-book/ch5-web/ch5-02-router.html

# 常用框架

部分框架对比：

[awesome-go-web-frameworks](https://github.com/speedwheel/awesome-go-web-frameworks)

[awesome-go-web-frameworks 翻译](https://mp.weixin.qq.com/s/N_mEhTEqEa7QofZLixjIaw)

- 重量级
  - [Beego](https://github.com/astaxie/beego)：用于 Go 编程语言的开源、高性能 Web 框架。
  - [Iris](https://github.com/kataras/iris)：社区驱动的 Web 框架。Webassembly、自动 HTTPS，MVC，会话，缓存，版本控制 API，Websocket，依赖注入等等。与标准库和第三方中间件软件包完全兼容。
  - [Revel](https://github.com/revel/revel)：Go 语言的高生产率，全栈式 Web 框架。
  - [gf](https://github.com/gogf/gf)：GoFrame 是 Go 的模块化，功能齐全且可用于生产环境的应用程序开发框架。
- 轻量级
  - [Gin](https://github.com/gin-gonic/gin)：使用 Go 编写的 HTTP Web 框架。它具有类似于 Martini （不再维护）的 API，并具有更好的性能，号称快 40 倍。
    - [教程](https://book.eddycjy.com/golang/gin/install.html)
  - [Echo](https://github.com/labstack/echo)：高性能，极简主义的 Go Web 框架。
    - [翻译](http://go-echo.org/ )
    - [教程](http://blog.studygolang.com/category/echo-%E7%B3%BB%E5%88%97/)
  - [Buffalo](https://github.com/gobuffalo/buffalo)：快速的 Web 开发框架。
  - [Macaron](https://github.com/go-macaron/macaron)：Go 高效且模块化的 Web 框架。
- 路由级
  - [mux](https://github.com/gorilla/mux)：强大的 HTTP 路由器和 URL 匹配器，用于构建 Go Web 服务器。
  - [fasthttp](https://github.com/valyala/fasthttp)：零内存分配的高性能 HTTP 库。 性能方面，它比 net/http 快 10 倍。
  - [httprouter](https://github.com/julienschmidt/httprouter)：轻量级的高性能 HTTP 请求路由器。
  - [net/http](http://docs.studygolang.com/pkg/net/http)：标准库 HTTP 包。
  - [chi](https://github.com/go-chi/chi)：轻便、惯用且可组合的路由器，用于构建 Go HTTP 服务。







