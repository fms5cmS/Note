RPC 是远程过程调用的简称，是分布式系统中不同节点间流行的通信方式。

Go 语言的 RPC 包的路径为 net/rpc。

标准库的 RPC 默认采用 Go 语言特有的 gob 编码，因此从其它语言调用 Go 语言实现的 RPC 服务将比较困难。在互联网的微服务时代，每个 RPC 以及服务的使用者都可能采用不同的编程语言，因此跨语言是互联网时代 RPC 的一个首要条件。得益于 RPC 的框架设计，Go 语言的 RPC 其实也是很容易实现跨语言支持的。

Go 语言的 RPC 框架有两个比较有特色的设计：一个是 RPC 数据打包时可以通过插件实现自定义的编码和解码；另一个是 RPC 建立在抽象的 `io.ReadWriteCloser` 接口之上的，我们可以将 RPC 架设在不同的通讯协议之上。

Go 语言的 RPC 框架支持异步调用，当返回结果的顺序和调用的顺序不一致时，可以通过 id 来识别对应的调用

# 基础

- 定义服务
  - **Go 语言的 RPC 服务的方法只能有两个可序列化的参数，其中第二个参数是指针类型，且返回一个 error 类型，同时必须是公开的方法**

```go
type HelloService struct{}

// Go 语言的 RPC 规则：
// 方法只能有两个可序列化的参数，其中第二个参数是指针类型，且返回一个 error 类型，同时必须是公开的方法。
func (p *HelloService) Hello(request string, reply *string) error {
	*reply = "hello " + request
	return nil
}
```

---

- 服务端

```go
func main() {
	// 将对象类型中所有满足 RPC 规则的对象方法注册为 RPC 函数，所有注册的方法会放在 “HelloService” 服务空间之下
	rpc.RegisterName("HelloService", new(service.HelloService))

	listener, err := net.Listen("tcp", ":1234")
	if err != nil {
		log.Fatal("ListenTCP error: ", err)
	}
	// 建立 TCP 连接(这里仅一个)
	conn, err := listener.Accept()
	if err != nil {
		log.Fatal("Accept error: ", err)
	}
	// 通过 rpc.ServeConn 函数在该 TCP 连接上为对方提供 RPC 服务
	rpc.ServeConn(conn)
}
```

---

- 客户端

```go
func main() {
  // 拨号 RPC 服务
  client, err := rpc.Dial("tcp", ":1234")
  if err != nil {
    log.Fatal("dialing: ", err)
  }

  var reply string
	// 调用具体的 RPC 方法，参数分别为：
	// RPC服务名.方法名, 传入 RPC 方法的两个参数
  err = client.Call("HelloService.Hello", "Yoshino", &reply)
  if err != nil {
    log.Fatal(err)
  }
  fmt.Println(reply)
}
```

# 重构

在涉及 RPC 的应用中，作为开发人员一般至少有三种角色：

- 服务端实现 RPC 方法的开发人员
- 客户端调用 RPC 方法的人员
- 制定服务端和客户端 RPC 接口规范的设计人员。

在前面的例子中为了简化将以上几种角色的工作全部放到了一起，虽然看似实现简单，但是不利于后期的维护和工作的切割。

- RPC 服务接口规范定义

```go
/*RPC 的服务接口规范*/

// 服务的名字(为了避免名字冲突，增加了包路径前缀)
const HelloServiceName = "base_refactor/server/Helloservice"

// 	服务要实现的详细的方法列表
type HelloServiceInterface interface {
	Hello(request string, reply *string) error
}

// 注册该类型服务的函数
// 可以避免命名服务名称的工作，同时也保证了传入的服务对象满足了RPC接口的定义
func RegisterHelloService(svc HelloServiceInterface) error {
	return rpc.RegisterName(HelloServiceName, svc)
}

/*
	为简化客户端调用 RPC 函数，对客户端进行简单包装
 */

type HelloServiceClient struct {
	*rpc.Client
}

func (p *HelloServiceClient) Hello(request string, reply *string) error {
	return p.Client.Call(HelloServiceName+"Hello", "hello", &reply)
}

var _ HelloServiceInterface = (*HelloServiceClient)(nil)

func DialHelloService(network, address string) (*HelloServiceClient, error) {
	c, err := rpc.Dial(network, address)
	if err != nil {
		return nil, err
	}
	return &HelloServiceClient{Client: c}, nil
}
```

---

- 客户端

```go
func main() {
	helloServiceClient, err := standard.DialHelloService("tcp", ":1234")
	if err != nil {
		log.Fatal("dialing ", err)
	}
	var reply string
	err = helloServiceClient.Hello("hello", &reply)
	if err != nil {
		log.Fatal(err)
	}
}
```

---

- 服务端

```go
type HelloService struct{}

func (p *HelloService) Hello(request string, reply *string) error {
	*reply = "hello: " + request
	return nil
}

func main() {
	standard.RegisterHelloService(new(HelloService))

	listener, err := net.Listen("tcp", ":1234")
	if err != nil {
		log.Fatal("listen TCP error ",err)
	}

	for{ // 支持多个TCP链接，然后为每个 TCP 链接提供 RPC 服务
		conn, err := listener.Accept()
		if err != nil {
			log.Fatal("accept error ",err)
		}
		go rpc.ServeConn(conn)
	}
}
```
