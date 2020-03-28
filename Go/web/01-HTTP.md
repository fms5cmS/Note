# HTTP 服务器

```go
func main() {
	// 使用 HandleFunc() 注册路由(将某个函数与一个路由规则进行绑定)，第二个参数是一个函数
	// HandleFunc() 则是将这个函数包装为实现了 http.Handler 接口的类型 http.HandlerFunc
	http.HandleFunc("/", hello)
	log.Println("Starting Http Server ...")
	// 启动 HTTP 服务器并监听发送到指定地址和端口号的 HTTP 请求
	// 如果是监听地址为 127.0.0.1 或 localhost，可简写为 ":4000"
	// 第二个参数通常为 nil，服务端会调用 http.DefaultServeMux 进行处理
	log.Fatal(http.ListenAndServe("localhost:4000",nil))
}

func hello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello World"))
}
```

服务端业务逻辑：

`HandleFunc(pattern string, handler func(ResponseWriter, *Request))`函数内部会将 handler 转为 `http.HandlerFunc` 类型(标准的 HTTP 请求处理器对象)，该类型实现了 `http.Handler` 接口：

```go
type Handler interface {
	ServeHTTP(ResponseWriter, *Request)
}
```

所以 `HandleFunc()` 函数实际是将第二个参数的函数转为实现了 `http.Handler` 接口的类型 `http.HandlerFunc`，那么自己创建一个实现了 `http.Handler` 接口的类型也是可行的！

```go
func main() {
	// 使用 Handle() 来注册路由，第二个参数是 http.Handler 接口
	http.Handle("/",&helloHandler{})
	log.Println("Starting Http Server ...")
	log.Fatal(http.ListenAndServe(":4000",nil))
}
// 创自定义一个实现了 http.Handler 接口的类型
type helloHandler struct {}

func (helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello World"))
}
```

## 服务复用器(ServeMux)

`Handle(pattern string, handler Handler)` 和 `HandleFunc(pattern string, handler func(ResponseWriter, *Request))` 这两个用于服务端业务逻辑的函数实际上是对 `http.DefaultServeMux`(该对象是一个 `http.ServeMux` 类型) 的一层封装，源码：

```go
func Handle(pattern string, handler Handler) {
  DefaultServeMux.Handle(pattern, handler)
}
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

`http.ServeMux` 是一个带有基本路由功能的服务复用器(Multiplexer)，注意，该类型也是一个 `http.Handler` 接口的实现类型。

可以使用 `http.NewServeMux()` 创建一个 `http.ServeMux`对象：

```go
func main() {
	mux := http.NewServeMux()
	mux.Handle("/",&helloHandler{})
	log.Println("Starting Http Server ...")
	log.Fatal(http.ListenAndServe(":4000",nil))
}
```

## 服务器对象(Server)

```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```

最开始提到，调用该函数时，第二个参数通常传入 `nil`，如果直接传入一个处理器(Handler)，就无法进行路由注册了。

从 `http.ListenAndServe()` 的源码可以看出，其底层创建了一个 `http.Server` 对象来使用，`http.Server` 该对象包含了很多选项，如监听地址、服务复用器和读写超时等等。使用 `http.Server` 改写之前的代码：

```go
func main() {
	mux := http.NewServeMux()
	mux.Handle("/", &helloHandler{})
	mux.HandleFunc("/timeout", func(w http.ResponseWriter, r *http.Request) {
		time.Sleep(2*time.Second)
		w.Write([]byte("Timeout"))
	})
	server := http.Server{
		Addr:         ":4000",
    // 用于处理程序响应 HTTP 请求，ServerMux 也是一个 Handler 接口的实现类
		Handler:      mux,
		// 允许写入的最大时间
		WriteTimeout: 2 * time.Second,
	}
	log.Println("Starting Http Server ...")
	log.Fatal(server.ListenAndServe())
}
```

上面的 `http.Server` 对象设置了写超时(WriteTimeout)，写超时包含的范围是当请求头被解析后直到响应完成，即从绑定的函数开始执行到结束为止，如果耗时超过这个值就会被认为超时，从而提前关闭与客户端的连接，被绑定的函数也就不会执行了，所以当访问 `http://localhost:4000/timeout` 时不会输出任何内容。

## 停止服务

在产生环境中面临的一个困难就是当需要更新服务端程序时需要重启服务，但此时可能有一部分请求进行到一半，如果强行中断这些请求可能会导致意外的结果。因此，开源社区提供了多种优雅停止的方案，如 facebookgo/grace。从 Go 1.8 版本开始，标准库开始支持原生的优雅停止方案，但必须使用自定义的 `http.Server` 对象，因为对应的 `Close` 方法无法通过其它途径调用。

```go
func main() {
	mux := http.NewServeMux()
	mux.Handle("/", &HelloHandler{})
	server := &http.Server{
		Addr:    ":4000",
		Handler: mux,
	}

	quit := make(chan os.Signal)
	signal.Notify(quit,os.Interrupt)
	go func() {
		<- quit
		if err:=server.Close();err!=nil{
			log.Fatal("Close server:", err)
		}
	}()
	log.Println("Starting HTTP server...")
	err := server.ListenAndServe()
	if err != nil {
		if err == http.ErrServerClosed {
			log.Print("Server closed under request")
		} else {
			log.Fatal("Server closed unexpected")
		}
	}
}
// HelloHandler 的代码...
```

程序会捕捉 `os.Interrupt`(Ctrl+C)信号，然后调用 `Close()` 方法告知服务器应停止接受新的请求并在处理完当前已接受的请求后关闭服务器。

标准库提供了一个特定的错误类型 `http.ErrServerClosed`，可用它判断确定服务器是正常关闭的还是意外关闭的。

## 处理 HTTPS 请求

处理 HTTPS 请求：`http.ListenAndServeTLS(addr, certFile, keyFile string, handler Handler)`

- 服务器上必须存在包含证书和与之匹配的私钥的相关文件
  - certFile 对应 SSL 证书文件存放路径
  - keyFile 对应证书私钥文件路径。
  - 如果证书是由证书颁发机构签署的， certFile 参数指定的路径必须是存放在服务器上的经由 CA 认证过的 SSL 证书
- 和 `http.ListenAndServe(addr string, handler Handler)`，区别在于只处理 HTTPS 请求

业务逻辑和 HTTP 请求的相同。

# HTTP 客户端

## 常用方法

请求资源：`http.Get(url string)`

从源码可看出，实际上调用的是`http.DefaultClient.Get(url)`：

```go
resp, err := http.Get("https://www.bilibili.com")
//必须对 resp.Body 进行关闭
defer resp.Body.Close()
```

---

以 POST 方式发送数据：`http.Post(url, contentType string, body io.Reader)`

- contentType 为将要 POST 数据的资源类型
- body 为数据的比特流：

从源码可看出，实际上调用的是`http.DefaultClient.Post(url, contentType, body)`

```go
resp, err := http.Post("http://example.com/upload", "image/jpeg", &imageDataBuf)
//必须对 resp.Body 进行关闭
defer resp.Body.Close()
```

---

- 只请求目标 URL 的头部信息：`http.Head(url string)`
  - 从源码可看出，实际上调用的是`http.DefaultClient.Head(url)`

## httputil

使用 httputil 简化工作：

```go
s, err := httputil.DumpResponse(resp, true)
//返回的 s 是一个 []byte
fmt.Printf("%s\n", s)
```

## http.Client

- 使用 `http.Client` 定制信息，如控制请求头等（下面的代码省略了错误的处理）

```go
request, err := http.NewRequest(http.MethodGet, "https://www.imooc.com", nil)
//这里通过设置 User-Agent 来模拟浏览器访问慕课网！！
request.Header.Add("User-Agent",
                   "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/75.0.3770.142 Safari/537.36")
//使用默认的 Client
resp, err := http.DefaultClient.Do(request)
defer resp.Body.Close()

s, err := httputil.DumpResponse(resp, true)
fmt.Printf("%s\n", s)
```

---

- 自定义 `http.Client`

`http.Client`类型的结构：

```go
type Client struct {
  //用于确定HTTP请求的创建机制,如果为空,将会使用 DefaultTransport
  Transport RoundTripper

  //定义重定向策略。如果不为空，客户端将在跟踪HTTP重定向前调用该函数
  //两个参数req和via分别为即将发起的请求和已经发起的所有请求，最早的已发起请求在最前面。
  //如果CheckRedirect返回错误，客户端将直接返回错误，不会再发起该请求。
  // 如果CheckRedirect为空，Client将采用一种确认策略，将在10个连续请求后终止
  CheckRedirect func(req *Request, via []*Request) error

  //如果Jar为空，Cookie将不会在请求中发送，并会在响应中被忽略。
  //一般用  http.SetCookie() 方法来设定 Cookie
  Jar CookieJar

  Timeout time.Duration
}
```

自定义：

```go
request, err := http.NewRequest(http.MethodGet, "https://www.imooc.com", nil)
request.Header.Add("User-Agent",
                   " Mozilla/5.0 (iPhone; CPU iPhone OS 11_0 like Mac OS X) AppleWebKit/604.1.38 (KHTML, like Gecko) Version/11.0 Mobile/15A372 Safari/604.1")

//自定义的 Client
client := http.Client{
  CheckRedirect: func(req *http.Request, via []*http.Request) error {
    //从程序运行结果可以看出有重定向
    fmt.Println("Redirect: ",req)
    return nil
  },
}

resp, err := client.Do(request)
defer resp.Body.Close()

s, err := httputil.DumpResponse(resp, true)
fmt.Printf("%s\n", s)
```

# http 服务器性能分析

- `import _ "net/http/pprof"`
  - `_`表示虽然这个文件没有用到，但会加载一些帮助程序
- 启动程序
- 访问 /debug/pprof
- 使用 `go tool pprof http://地址/debug/pprof/要分析的内容` 分析性能
  - 如：查看 30s 内 CPU 的情况：`go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30`
  - 查看堆的情况：`go tool pprof http://localhost:6060/debug/pprof/heap`
  - 其他见 net/http/pprof.pprof.go 文件
- 上面异步出来结果后，输入`web`可以在浏览器查看结果

# 乱码问题

从网页上获取的 HTML 文件的编码可能不是 UTF-8 的，而 Go 的字符集默认为 UTF-8，这种情况下会出现乱码。那么，如何解决乱码问题呢？

- 下载库：

  ```shell
  gopm get -g -v golang.org/x/text
  gopm get -g -v golang.org/x/net/html
  ```

如果仅使用 text 库：

```go
utf8Reader := transform.NewReader(resp.Body,
                                  simplifiedchinese.GBK.NewDecoder())
all, err := ioutil.ReadAll(utf8Reader)
```

这种方式的通用性较差，所以通常会结合 html 库来自动检测 html 文件的编码：

```go
//对 charset.DetermineEncoding 进行了包装
func determineEncoding(r io.Reader) encoding.Encoding {
	//如果直接从 r 中读取前 1024 bytes 的话，这 1024 bytes 就没办法再读了，所以这里使用 bufio 包装 r 以后 Peek
	bytes, err := bufio.NewReader(r).Peek(1024)
	if err != nil {
		panic(err)
	}
	encoding, _, _ := charset.DetermineEncoding(bytes, "")
	return encoding
}
```

```go
func main()  {
  //获取 HTML 的编码，对 charset.DetermineEncoding() 进行了包装，见上面
  e := determineEncoding(resp.Body)
  //将其转为 UTF-8
  utf8Reader := transform.NewReader(resp.Body, e.NewDecoder())
  all, err := ioutil.ReadAll(utf8Reader)
}
```
