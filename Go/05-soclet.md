注意：Go 的 Socket 编程 API 在底层获取的是一个非阻塞式的 socket 实例，即在该实例上的数据读取操作也都是非阻塞式的。部分 socket 编程 API 调用起来像是阻塞式的，但应明确，它在底层使用的是非阻塞式的 socket 接口。



# 服务端

## net.Listen()

`func Listen(network, address string) (Listener, error)` 用于获取监听器。

- 参数

第一个参数指定监听协议，第二个 参数指定监听的地址（格式为：``host:port`）。

可以使用字符串常量来标识协议，Go 中使用的协议字符串字面量有：

| 字面量     | socket 协议 | 备注                                                 |
| ---------- | ----------- | ---------------------------------------------------- |
| tcp        | TCP         | 无                                                   |
| tcp4       | TCP         | 仅支持 IPv4                                          |
| tcp6       | TCP         | 仅支持 IPv6                                          |
| udp        | UDP         | 无                                                   |
| udp4       | UDP         | 仅支持 IPv4                                          |
| udp6       | UDP         | 仅支持 IPv6                                          |
| unix       | 有效        | 通信域AF_UNIX且类型SOCK_STREAM时内核使用的的默认协议 |
| unixgram   | 有效        | AF_UNIX 且类型 SOCK_DGRAM 时内核使用的的默认协议     |
| unixpacket | 有效        | AF_UNIX 且类型 SOCK_SEQPACKET 时内核使用的的默认协议 |

注意：这个参数代表的值必须是面向流的协议。TCP、SCTP 都属于面向流的传输层协议。所以，`net.Listen()` 的第一个参数的值必须是 tcp、tcp4、tcp6、unix、unixpacket 的其中之一。

对于基于 TCP 协议的 socket 而言，`net.Listen()` 的第二个参数的值标识当前程序在网络中的标识。

- 返回值

`net.Listen()` 函数会返回一个 `net.Listener` 类型的监听器和 `error` 类型的值。如果第二个参数使用主机名，则程序会先通过 DNS 找到对应的 IP，如果该主机名没有在 DNS 中注册，会报错。



## listener.Accept()

对指定网络地址使用指定协议监听，进行了必要的错误检查后就可以等待客户端的连接请求了。

调用监听器的 `Accept()` 方法，流程会被阻塞，直到某个客户端程序与当前程序建立 TCP 连接。该方法会返回两个结果值：代表了当前 TCP 连接的 `net.Conn` 类型的值、`error` 类型的值。



# 客户端

## net.Dial()

`func Dial(network, address string) (Conn, error) ` 向指定的网络地址发送链接建立申请。

- 参数

第一个参数类似 `Listen()` 的第一个参数，但却拥有更多的可选值，因为在发送数据前不一定要先建立连接。如 UDP、IP 协议都是面向无连接型的协议，所以 udp、udp4、udp6、ip、ip4、ip6 都可以作为第一个参数值。

第二个参数与 `Listen()` 的第二个参数相同，如果要与前面开始监听的服务端程序连接的话，这个参数的值就是服务端的地址。

- 返回值

`net.Dail()` 函数回返回一个 `net.Conn` 类型的值和 `error` 类型的值。如果参数值不合法，则第二个结果值非 `nil`。此外，对于 TCP 协议的连接请求而言，当远程地址上没有正在监听的程序时，也会使 `net.Dail` 返回一个非 `nil` 的错误。



## net.DialTimeout()

使用 `func DialTimeout(network, address string, timeout time.Duration) (Conn, error)` 设置超时时间。



## net.Conn

当客户端和服务端成功建立连接后，服务端和客户端都会得到一个 `net.Conn` 类型的值，两端可以分别使用各自的这个值来交换数据。该接口类型共有 8 个方法，定义了在一个连接上做的所有事情。



## Read()

`Read(b []byte) (n int, err error)` 从 socket 的接收缓冲区中读取数据。参数 b 是用于存放接收到的数据。返回值 n 代表实际读取到的字节数。

如果返回的是 `io.EOF` 错误，则终止后续的数据读取操作，并关闭该 TCP 连接。

```go
	var dataBuf bytes.Buffer  //存储接收到的所有数据
	b := make([]byte, 10)
	for{
		n, err := conn.Read(b)
		if err!=nil{
			if err == io.EOF{
				fmt.Println("The connection is closed.")
				conn.Close()
			} else {
				fmt.Printf("Read error: %s\n",err)
			}
			break
		}
		dataBuf.Write(b[:n]) //没有错误时，将读取到的数据追加到 dataBuf 中
	}
```

该方法实际是 `io.Reader` 接口中唯一的方法，所以 `net.Conn` 也是 `Reader` 接口的一个实现类型。可以使用 `bufio.NewReader()` 对 conn 进行包装：`reader := bufio.NewReader(conn)`，然后就可以使用 reader 的方法了。



## Write()

`Write(b []byte) (n int, err error)` 用于向 socket 的发送缓冲区写入数据。该方法是 `io.Writer` 接口的中唯一的方法，所以也可以使用 `bufio.NewWriter()` 对 conn 进行包装。

注意：

在向缓冲区写入数据后，要调用 `Flush()` 方法以保证其中所有的数据都真正写入了它代理的对象（这里是由 conn 变量代表的 TCP 连接）。

```go
	writer := bufio.NewWriter(conn)
	writer.WriteString("bx")
	writer.Flush()
```

使用 `bufio.NewWriter()` 对 conn 进行包装后，缓冲区的容量默认为 4096 字节，为避免参数值的数据长度超出容量，可以调用 `bufio.NewWriterSize()` 来初始化一个缓冲写入器，自定义缓冲区容量。



## Close()

关闭当前连接，该方法不接受参数，返回一个 `error` 类型的值。在关闭连接后，如果继续调用 conn 的 `Read()`、`Write()` 方法，会立即返回错误。

如果调用 `Close()` 方法时，`Read()`、`Write()` 方法正在被调用且还未执行结束，这两个方法也会立即结束执行，并返回错误。即使它们正处于阻塞状态，也会这样。



## 地址相关方法

`LocalAddr() Addr`，返回本地地址；`RemoteAddr() Addr`，返回远程地址。

`net.Addr` 是接口类型，有 `Network() string`、`String() string` 两个方法，前者返回当前连接使用的协议名称，后者返回相应的一个地址。

对于客户端程序，如果在与服务端程序通信时没有指定本地地址，则 `LocalAddr()` 会返回操作系统内核为客户端程序分配的网络地址。



## 操作超时设置

- `SetDeadline(t time.Time) error`，设置当前连接上的 I/O 操作（包括但不限于读写）的超时时间；
  - 注意，对该语句之后的所有 I/O 操作的总时间进行设置

```go
b := make([]byte,10)
conn.SetDeadLine(time.Now().Add(2*time.Second))
//如果某次读操作执行时，发现已经超时，则操作立即失败，且后续迭代的读操作也都会失败
//所以需要在每次 I/O 操作执行前进行设置，可以将 SetDeadline 语句放在循环体内！！
for{
  n,err:= conn.Read(b)
  //...
}
```

如何取消超时时间的设置呢？同样使用 `SetDeadline()` 方法，将参数设置为 `time.Time{}` 即可！

- `SetReadDeadline(t time.Time) error`，针对当前连接上的读操作设置超时时间
- `SetWriteDeadline(t time.Time) error`，针对当前连接上的写操作设置超时时间
  - 注意，写操作超时，并不代表写操作完全没有成功。部分数据可能已经写入到 socket 的发送缓冲区了