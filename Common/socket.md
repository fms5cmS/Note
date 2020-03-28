Linux 系统中，有一个名为 socket 的系统调用：`int socket(int domain,int type,int protocol)`，其功能是创建一个 Socket 实例，参数分别为该 Socket 的通信域、类型、所用协议。

Socket 的通信域：

| 通信域   | 含义   | 地址形式         | 通讯范围                                      |
| -------- | ------ | ---------------- | --------------------------------------------- |
| AF_INET  | IPv4域 | IPv4地址，端口号 | IPv4 的网络中的任意两台计算机上的两个应用程序 |
| AF_INET6 | IPv6域 | IPv6地址，端口号 | IPv6 的网络中的任意两台计算机上的两个应用程序 |
| AF_UNIX  | Unix域 | 路径名称         | 同一台计算机上的两个应用程序                  |

Socket 的类型：SOCK_STREAM、SOCK_DGRAM、面向更底层的 SOCK_RAW、针对某个新兴数据传输技术的 SOCK_SEQPACKET。

| 特性       | SOCK_DGRAM | SOCK_RAW | SOCK_SEQPACKET | SOCK_STREAM |
| ---------- | ---------- | -------- | -------------- | ----------- |
| 数据形式   | 数据报     | 数据报   | 字节流         | 字节流      |
| 数据边界   | 有         | 有       | 有             | 无          |
| 逻辑连接   | 无         | 无       | 有             | 有          |
| 数据有序性 | 不保证     | 不保证   | 能保证         | 能保证      |
| 传输可靠性 | 不具备     | 不具备   | 具备           | 具备        |

调用系统调用 socket 时，一般把 0 作为第三个参数，含义：由操作系统内核根据第一二个参数值自行决定 Socket 使用的协议，Socket 协议默认选择：

|          | SOCK_DGRAM | SOCK_RAW | SOCK_SEQPACKET | SOCK_STREAM |
| -------- | ---------- | -------- | -------------- | ----------- |
| AF_INET  | UDP        | IPv4     | SCTP           | TCP 或 SCTP |
| AF_INET6 | UDP        | IPv6     | SCTP           | TCP 或 SCTP |
| AF_UNIX  | 有效       | 无效     | 有效           | 有效        |

无效标识该通信域与类型的组合是不合法的。

TCP：Transmission Control Protocol 传输控制协议

UDP：User Datagram Protocol 用户数据报协议

SCTP：Stream Control Transmission Protocol 流控制传输协议

在没有发生错误的情况下，系统调用 socket 会返回一个 `int` 类型的值，该值是作为 socket 唯一标识符的文件描述符。得到该标识符号，就可以调用其他系统调用来进行各种相关操作了，如绑定、监听端口、发送/接收数据、关闭 Socket 实例等。

