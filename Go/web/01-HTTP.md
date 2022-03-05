# HTTP 服务器

```go
func main() {
	http.HandleFunc("/", hello)
	log.Println("Starting Http Server ...")
	// 如果是监听地址为 127.0.0.1 或 localhost，可简写为 ":4000"
	log.Fatal(http.ListenAndServe("localhost:4000",nil))
}

func hello(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello World"))
}
```

## http.Handler 接口

- `http.Handler` 接口

```go
type Handler interface {
 ServeHTTP(ResponseWriter, *Request)
}
```

- 服务复用器(Multiplexer) `http.ServeMux`，带有基本的路由注册功能

```go
type ServeMux struct {/* 这里没有列出具体的字段 */}
func (mux *ServeMux) ServeHTTP(w ResponseWriter, r *Request){/* ... */}
// 其他方法...
// 对外提供一个默认的 ServeMux 来使用
var DefaultServeMux = &defaultServeMux
var defaultServeMux ServeMux
```

- 标准的 http 请求处理器类型 `http.HandleFunc`

```go
type HandlerFunc func(ResponseWriter, *Request)
// ServeHTTP calls f(w, r).
func (f HandlerFunc) ServeHTTP(w ResponseWriter, r *Request) {
	f(w, r)
}
```

## 注册路由

方式一：

```go
func HandleFunc(pattern string, handler func(ResponseWriter, *Request)) {
	DefaultServeMux.HandleFunc(pattern, handler)
}
```

默认的 `ServeMux` 会调用 `ServeMux.HandleFunc()`方法：

```go
func (mux *ServeMux) HandleFunc(pattern string,
                                handler func(ResponseWriter, *Request)) {
	if handler == nil {
		panic("http: nil handler")
	}
  // 先将 handler 转为 HandlerFunc 类型(Handler 接口的实现类型)
  // 然后将 pattern 和 Handler 接口的实现类型关联
	mux.Handle(pattern, HandlerFunc(handler))
}
```

示例见最开始的部分。

---

方式二：

```go
func Handle(pattern string, handler Handler) {
  DefaultServeMux.Handle(pattern, handler)
}
```

此时需要自定义一个实现了 `http.Handler` 接口的实现类，示例：

```go
func main() {
	http.Handle("/",&helloHandler{})
	// ...
}
// 自定义一个实现了 http.Handler 接口的类型
type helloHandler struct {}

func (helloHandler) ServeHTTP(w http.ResponseWriter, r *http.Request) {
	w.Write([]byte("Hello World"))
}
```

---

方式三：

以上两种方式都是调用了 http 包提供的默认的 `ServeMux` 来注册路由的，可以使用 http.NewServeMux() 创建一个 `http.ServeMux` 对象然后注册路由：

```go
func main() {
	mux := http.NewServeMux()
	mux.Handle("/",&helloHandler{})
	log.Println("Starting Http Server ...")
	log.Fatal(http.ListenAndServe(":4000",nil))
}
```

## 启动服务

`http.ListenAndServe()` 函数启动 HTTP 服务器并监听发送到指定地址和端口号的 HTTP 请求：

```go
func ListenAndServe(addr string, handler Handler) error {
	server := &Server{Addr: addr, Handler: handler}
	return server.ListenAndServe()
}
```

第二个参数通常为 `nil`，服务端会调用 `http.DefaultServeMux` 进行处理，如果直接传入一个处理器(`Handler`)，就无法进行路由注册了。

`http.ListenAndServe()`会创建一个 `http.Server` 对象(包含很多选项，如监听地址、服务复用器和读写超时等)，然后调用该对象的 `ListenServe()` 方法，也可以自定义一个 `http.Server`对象来启动服务：

```go
func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/timeout", func(w http.ResponseWriter, r *http.Request) {
		time.Sleep(2*time.Second)
		w.Write([]byte("Timeout"))
	})
	server := http.Server{
		Addr:         ":4000",
		Handler:      mux,  // 用于处理程序响应 HTTP 请求
		WriteTimeout: 2 * time.Second, // 允许写入的最大时间
	}
	log.Println("Starting Http Server ...")
	log.Fatal(server.ListenAndServe())
}
```

上面的 `http.Server` 对象设置了写超时(WriteTimeout)，写超时包含的范围是当请求头被解析后直到响应完成，即从绑定的函数开始执行到结束为止，如果耗时超过这个值就会被认为超时，从而提前关闭与客户端的连接，被绑定的函数也就不会执行了，所以当访问 `http://localhost:4000/timeout` 时不会输出任何内容。

## 停止服务

如果暴力地停止服务，那么已经发送给服务端的请求，来不及处理服务就被删掉了，就会造成这部分请求失败，服务就会有波动。所以服务在退出的时候，都需要先停掉流量再停止服务，这样服务的关闭才会更平滑。比如，消息队列处理器就是要将所有已经从消息队列中读出的消息，处理完之后才能退出。

开源社区提供了多种优雅停止的方案，如 facebookgo/grace。从 Go 1.8 开始，标准库开始支持原生的优雅停止方案，但必须使用自定义的 `http.Server` 对象，因为对应的 `Shutdown()` 方法无法通过其它途径调用。

```go
func main() {
	mux := http.NewServeMux()
	mux.HandleFunc("/test", func(w http.ResponseWriter, r *http.Request) {
		time.Sleep(10 * time.Second)
		w.Write([]byte("this is response for test"))
	})
	server := &http.Server{
		Addr:    ":8080",
		Handler: mux,
	}

	go func() {
		if err := server.ListenAndServe(); err != nil && !errors.Is(err, http.ErrServerClosed) {
			log.Fatalf("listenAndServe failed: %v", err)
		}
	}()
	// 等待中断信号
	quit := make(chan os.Signal, 1)
	signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
	<-quit
	log.Println("Shutdown server...")
	// 最大时间控制，通知该服务端它有 5s 的时间处理原有的请求
	ctx, cancel := context.WithTimeout(context.Background(), 10*time.Second)
	defer cancel()
	if err := server.Shutdown(ctx); err != nil {
		log.Fatal("Server forced to shutdown: ", err)
	}
	log.Println("Server exiting")
}
```

程序会捕捉 `os.Interrupt`(Ctrl+C)信号，然后调用 `Shutdown()` 方法告知服务器应停止接受新的请求并在处理完当前已接受的请求后关闭服务器。

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

`http.Client`：

```go
type Client struct {
  // 用于确定HTTP请求的创建机制,如果为空,将会使用 DefaultTransport
  Transport RoundTripper

  // 定义重定向策略。如果不为空，客户端将在跟踪HTTP重定向前调用该函数
  // 两个参数req和via分别为即将发起的请求和已经发起的所有请求，最早的已发起请求在最前面。
  // 如果CheckRedirect返回错误，客户端将直接返回错误，不会再发起该请求。
  // 如果CheckRedirect为空，Client将采用一种确认策略，将在10个连续请求后终止
  CheckRedirect func(req *Request, via []*Request) error

  // 如果 Jar 为空，Cookie 将不会在请求中发送，并会在响应中被忽略。
  // 一般用  http.SetCookie() 方法来设定 Cookie
  Jar CookieJar

  Timeout time.Duration
}
// 对外提供的默认客户端
var DefaultClient = &Client{}
```

## 常用方法

GET 方式请求资源：`http.Get(url string)`

```go
// 底层实际调用 http.DefaultClient.Get(url)
resp, err := http.Get("https://www.bilibili.com")
// 必须对 resp.Body 进行关闭
defer resp.Body.Close()
```

---

POST 方式发送数据：`http.Post(url, contentType string, body io.Reader)`

- contentType 为将要 POST 数据的资源类型
- body 为数据的比特流

```go
// 底层实际调用 http.DefaultClient.Post(url, contentType, body)
resp, err := http.Post("http://example.com/upload", "image/jpeg", &imageDataBuf)
//必须对 resp.Body 进行关闭
defer resp.Body.Close()
```

---

HEAD 方式获取头部信息：`http.Head(url string)`，底层实际调用的是 `http.DefaultClient.Head(url)`

## httputil

使用 httputil 简化工作：

```go
s, err := httputil.DumpResponse(resp, true)
//返回的 s 是一个 []byte
fmt.Printf("%s\n", s)
```

## http.Client

- 使用 `http.Client` 定制信息，如控制请求头等（这里省略了错误的处理）

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
utf8Reader := transform.NewReader(resp.Body, simplifiedchinese.GBK.NewDecoder())
all, err := ioutil.ReadAll(utf8Reader)
```

这种方式的通用性较差，所以通常会结合 html 库来自动检测 html 文件的编码：

```go
func main()  {
  // 获取 HTML 的编码
  e := determineEncoding(resp.Body)
  // 对 resp.Body 包装，使其读取编码格式为 e.NewDecodeer()
  htmlReader := transform.NewReader(resp.Body, e.NewDecoder())
  all, err := ioutil.ReadAll(htmlReader)
}

// 对 charset.DetermineEncoding 进行了包装，这里通过响应的前 1024 字节来确定编码
func determineEncoding(r io.Reader) encoding.Encoding {
	// 如果直接从 r 中读取前 1024 bytes 的话，这 1024 bytes 就没办法再读了，所以这里使用 bufio 包装 r 以后 Peek
	bytes, err := bufio.NewReader(r).Peek(1024)
	if err != nil {
		panic(err)
	}
	encoding, _, _ := charset.DetermineEncoding(bytes, "")
	return encoding
}
```
