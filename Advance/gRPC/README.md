RPC(Remote Procedure Call，远程方法调用)是解决分布式系统通信问题的一大利器。

其实 RPC 就是把拦截到的方法参数，转成可以在网络中传输的二进制，并保证在服务提供方能正确地还原出语义，最终实现像调用本地一样地调用远程的目的。

- 屏蔽远程调用跟本地调用的区别，让我们感觉就是调用项目内的方法；
- 隐藏底层网络通信的复杂性，让我们更专注于业务逻辑。

RPC 框架能够帮助我们解决系统拆分后的通信问题，并且能让我们像调用本地一样去调用远程方法。

# RPC基础

RPC 一般默认采用 TCP 来传输。

网络传输的数据必须是二进制数据，但调用方请求的出入参数都是对象。对象肯定没法直接在网络中传输，需要提前把它转为可传输的二进制，且要求转换算法是可逆的，这一过程一般叫做序列化。

调用方持续地把请求参数序列化成二进制后，经过 TCP 传输给了服务提供方。服务提供方从 TCP 通道里面收到二进制数据，那如何知道一个请求的数据到哪里结束，是一个什么类型的请求呢？

这就需要用到协议。数据格式的约定内容叫做“协议”。大多数的协议会分成两部分，分别是数据头和消息体。数据头一般用于身份识别，包括协议标识、数据大小、请求类型、序列化类型等信息；消息体主要是请求的业务参数信息和扩展属性等。

根据协议格式，服务提供方就可以正确地从二进制数据中分割出不同的请求来，同时根据请求类型和序列化类型，把二进制的消息体逆向还原成请求对象。这个过程叫作“反序列化”。

服务提供方再根据反序列化出来的请求对象找到对应的实现类，完成真正的方法调用，然后把执行结果序列化后，回写到对应的 TCP 通道里面。调用方获取到应答的数据包后，再反序列化成应答对象，这样调用方就完成了一次 RPC 调用。

如何简化 API，屏蔽掉 RPC 细节，让使用方仅需关注业务接口，像调用本地一样调用远程？

- 服务提供者给出业务接口声明；
- 在调用方程序中，RPC 框架根据调用的服务接口提前生成动态代理实现类，并通过依赖注入等技术注入到声明了该接口的相关业务逻辑中
  - 该代理实现类会拦截所有方法调用，在提供的方法处理逻辑中完成一整套远程调用，并将远程调用结果返回给调用方。这样调用方在调用远程方法时就获得了像调用本地接口一样的体验。

RPC 框架是为了让远程服务调用更简单、透明，会屏蔽底层的传输方式（TCP 或 UDP）、序列化方式（XML/JSON/二进制）和通信细节。

# gRPC

gRPC 是由 Google 开发并且开源的一款高性能、跨语言的 RPC 框架：

- 基于 IDL 文件定义服务，通过 proto3 工具生成指定语言的数据结构、服务端接口及客户端 Stub；
- 通信协议基于标准的 HTTP/2 设计，支持双向流、消息头压缩、单 TCP 的多路复用、服务端推送等特性，这使得 gRPC 在移动端设备上更省电、节省网络流量；
- 序列化支持 PB（Protocol Buffer，语言无关的高性能序列化框架）和 JSON。

gRPC 与 Go 标准库的 RPC 框架不同，gRPC生成的接口并不支持异步调用。可以在多个 goroutine 之间安全地共享 gRPC 底层的 HTTP/2 链接，因此可以通过在另一个 goroutine 阻塞调用的方式模拟异步调用。

RPC 是远程函数调用，所以每次调用的函数参数、返回值不能太大，否则会严重影响每次调用的响应时间。传统 RPC 模式也不适用于对时间不确定的订阅和发布模式。为此，gRPC 框架针对服务器端和客户端分别提供了流特性。

```shell
go install google.golang.org/protobuf/cmd/protoc-gen-go@v1.26
go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@v1.1
# 项目中导入库
go get -u google.golang.org/grpc
```

hello.proto：

```protobuf
// 声明当前使用的语法为 proto3，不写则 protoc 默认使用的 proto2 语法
syntax = "proto3";
// 指定该 proto 文件生成的 Go 文件的完整（相对于当前项目）导入路径，可以不写，建议有
option go_package = "learn_grpc/hello/protos";

package protos;

service Greeter{
  // 定义 RPC 方法，指定入参和返回值
  rpc SatHello (HelloRequest) returns (HelloReply);
}

message HelloRequest{
  // Protobuf 编码是通过成员的唯一编号来绑定对应数据的
  string name = 1;
}

message HelloReply{
  string message = 1;
}
```

命令行执行：

```shell
protoc --go_out=. --go_opt=paths=source_relative \
	--go-grpc_out=. --go-grpc_opt=paths=source_relative \
	protos/hello.proto
```

会自动生成 hello.pb.go 和 hello_grpc.pb.go 两个文件，文件中分别包含：用于填充、序列化、检索 `HelloRequest`、`HelloReply` 消息类型的代码；生成的客户端、服务端接口代码。

尽管客户端、服务端代码自动生成了，但仍需要实现并调用方法。

服务端：

```go
// 实现 proto.GreeterServer 接口
type server struct {
	// 必须嵌入 proto.UnimplementedGreeterServer 结构体
	protos.UnimplementedGreeterServer
}

func (s *server) SatHello(ctx context.Context, req *protos.HelloRequest) (*protos.HelloReply, error) {
	reply := &protos.HelloReply{Message: "hello: " + req.Name}
	return reply, nil
}

func main() {
	lis, err := net.Listen("tcp", ":1234")
	if err != nil {
		log.Fatalf("failed to listen: %v", err)
	}
	// 构建 gRPC 服务对象
	s := grpc.NewServer()
	// 使用 protoc 工具生成的函数注册服务
	protos.RegisterGreeterServer(s, &server{})
	// 指定在监听端口提供 gRPC 服务
	if err := s.Serve(lis); err != nil {
		log.Fatalf("failed to serve: %v", err)
	}
}
```

客户端：

```go
func main() {
	// 与 gRPC 服务建立连接
	conn, err := grpc.Dial("localhost:1234", grpc.WithInsecure())
	if err != nil {
		log.Fatalf("did not connect: %v", err)
	}
	defer conn.Close()

	// 基于已建立的连接构造客户端对象
	c := protos.NewGreeterClient(conn)
	ctx, cancel := context.WithTimeout(context.Background(), time.Second)
	defer cancel()
	// 调用 gRPC 服务提供的方法
	reply, err := c.SayHello(ctx, &protos.HelloRequest{Name: "fms5cmS"})
	if err != nil {
		log.Fatalf("could not greet: %v", err)
	}
	log.Printf("Greeting: %s", reply.GetMessage())
}
```

# Question

Q：gRPC 为什么是基于 HTTP/2 协议的？

许多客户端要通过 HTTP 代理来访问网络，gRPC 全部使用 HTTP/2 实现，等到代理开始支持 HTTP/2 后就能透明转发 gRPC 数据。而且负责负载均衡等的反响代理也能无缝兼容 gRPC 了。

