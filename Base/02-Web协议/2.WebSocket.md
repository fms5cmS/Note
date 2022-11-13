
WebSocket 解决了什么问题？

当服务器端的资源经常更新的时候，客户端需要尽量及时地了解到这些更新，展示给用户，如果是 HTTP1.1，它通常会开启 AJAX 请求，问一下服务端有更新吗，如果没有更新再返回，再加一个定时器，通过定时器反复地去轮询服务端上相应的资源有没有被更新。这个模型下有很多问题，比如长时间不更新的情况下，反复地去询问服务器，会对服务器造成巨大的压力，对网络也会有很大的消耗。实际上，当定时的时间比较大的情况下，反而服务器上有更新了，我们的客户端可能还需要等待定时器到达以后才有可能获知，这个信息也不是很及时的被获取到。

WebSocket可以反向通知，WebSocket 通常向服务器订阅一类消息，服务器只要发现这类消息有更新就会不停地去通知客户端。

WebSocket 实现了服务器端的消息推送。

Chrome 中查看 WebSocket 的请求，类型选 WS，或属性过滤 `is: running`。

消息颜色
- 发送至服务器的文本消息为浅绿色;
- 接收到的文本消息为白色;
- WebSocket 操作码为浅黄色;
- 错误为浅红色。

# 约束Or对比

- 为了实时性与可伸缩性，而牺牲了简单性。

HTTP 协议，系统可以任意增加 Server 端，Client 的多个请求就是与负载均衡的多个连接，请求通过负载均衡分发给不同的 Server；

WS 协议，Client 的多个请求与负载均衡只有一个连接，为了比较好的可伸缩性，让多个 Server 服务于一个长连接，通常需要比较复杂的架构设计：会分为接入层和实现层，两层之间还会有一个消息分发系统（如 Kafka、RabbitMQ）。负载均衡将请求分发给不同接入层，接入层做比较简单的事，如协议转换，将 WS 协议转为消息分发系统可识别的协议，之后消息分发系统再将消息分发给实现层来处理。

- 网络效率与无状态：请求2 基于请求1

WS 协议，如果要做到 Client 上多个请求（仅一个连接）是无状态的，那么就需要每次传递大量重复消息。所以通常使后续请求基于前面请求（如请求1 是 html，请求2 是 css，1 中携带了 user_agent，2 就不用携带了），但是这样一来简单性和可见性就比较差了。

上面提到的“基于”是指业务依赖，如请求1是登录，请求2是用户信息列表展示。

- 长连接的心跳保持较为复杂

HTTP 长连接只能基于简单的超时(常见为 65 秒) ；
WebSocket 连接基于 ping/pong 心跳机制维持。

- 兼容 HTTP

默认使用 80 或者 443 端口
协议升级

# 设计哲学:在 Web 约束下暴露 TCP 给上层

- 元数据去哪儿了？
  - HTTP 协议头部会存放元数据
  - 由 WebSocket 上传输的应用层存放元数据
- WS 基于帧，而不是基于流（HTTP、TCP）
  - 每一帧要么承载字符数据，要么承载二进制数据
- 基于浏览器的同源策略模型(非浏览器无效)
  - 也可以使用 Access-Control-Allow-Origin 等头部
- 基于 URI、子协议支持同主机同端口上的多个服务

# 协议格式

WS 协议最基本的单位：帧。

## 帧类型

- 持续帧
  - 0，继续前一帧
- 非控制帧
  - 1:文本帧(UTF8)
  - 2:二进制帧
  - 3-7:为非控制帧保留
- 控制帧
  - 8:关闭帧
  - 9:心跳帧 ping
  - A:心跳帧 pong
  - B-F:为控制帧保留

# HTTP->WS

## URI 格式

ws-URI = `"ws:" "//" host [ ":" port ] path [ "?" query ]`，默认 port 端口 80
wss-URI = `"wss:" "//" host [ ":" port ] path [ "?" query ]`，默认 port 端口 443


客户端提供信息
- host 与 port，主机名与端口
- shema，是否基于 SSL
- 访问资源，URI
- 握手随机数，Sec-WebSocket-Key
- (可选)选择子协议，Sec-WebSocket-Protocol
- (可选)扩展协议，Sec-WebSocket-Extensions
- (可选)CORS 跨域，Origin

请求示例：

```shell
GET /?encoding=text HTTP/1.1                       # 必须，HTTP/1.0 不行
Host: websocket.taohui.tech                        # 必须
Accept-Encoding: gzip, deflate 
Sec-WebSocket-Version: 13                          # 必须，目前都是 13
Origin: http://www.websocket.org                   # 跨域信息
Sec-WebSocket-Extensions: permessage-deflate 
Sec-WebSocket-Key: c3SkgVxVCDhVCp69PJFf3A==        # 必须，随机数
Connection: keep-alive, Upgrade                    # 必须，声明要升级
Pragma: no-cache 
Cache-Control: no-cache 
Upgrade: websocket                                 # 必须
```

响应示例

```shell
HTTP/1.1 101 Web Socket Protocol Handshake               # 必须返回 101
Server: openresty/1.13.6.2
Date: Mon, 10 Dec 2018 08:14:29 GMT
Connection: upgrade                                      # 必须返回 upgrade
Access-Control-Allow-Credentials: true                   # Access-Control-Allow-xxx 的这几个都是跨域信息
Access-Control-Allow-Headers: content-type 
Access-Control-Allow-Headers: authorization 
Access-Control-Allow-Headers: x-websocket-extensions 
Access-Control-Allow-Headers: x-websocket-version 
Access-Control-Allow-Headers: x-websocket-protocol 
Access-Control-Allow-Origin: http://www.websocket.org 
Sec-WebSocket-Accept: yA9O5xGLp8SbwCV//OepMPw7pEI=       # 必须，Server 根据 Client 携带的随机数生成
Upgrade: websocket                                       # 必须
```

# todo...
