Go 的 runtime 包括了：

- Scheduler：调度器管理所有的 G、M、P，在后台执行循环调度
- Netpoll：网络轮询负责管理网络 FD 相关的读写、就绪事件
- Memory Management：当代码需要内存时，负责内存分配工作
- Garbage Collector：当内存不再需要时，负责回收内存

m0 是 Go 程序启动后创建的第一个线程。

- G：goroutine，计算任务，由需要执行的代码及其上下文组成，上下文包括 当前代码位置、栈顶、栈底地址、状态等
  - `go func` --> runtime.newproc
- M：machine，系统线程，执行实体，想要在 CPU 上执行代码，必须有线程，通过系统调用 clone 创建
  - 按需创建 --> runtime.newm --> clone
  - M 执行用户逻辑时，使用普通 g 的栈
  - M 执行 runtime 调度代码时，切换到 m.g0
  - M 执行信号处理时，切换到 gsignal 栈
- P：processor，虚拟处理器，M 必须获得 P 才能执行代码，否则必须陷入休眠（后台监控线程除外）
  - runtime.schedinit --> runtime.procresize
  - 有一个 schedtick 的计数器用来记录循环调度（schedule -> execute -> gogo -> goexit -> schedule...这里是消费端的四个函数）的次数，每次执行到 runtime.execute 次数会加一

# 生产-消费流程

Go 的调度流程本质上是一个生产-消费流程。

多级队列减少锁竞争：生产和消费的优先级都是从上往下

- 本地队列：
  - runnext
  - local runnable queue
- 全局队列：global runnable queue

生产端：

```go
package main

imoprt "runtime"

func main() {
    runtime.GOMAXPROCS(1)
    for i := 0; i < 9; i++ {
        i := i 
        go func() {
            println(i)
        }()
    }
    select{}
}
```

```shell
>> dlv debug main.go
>> b main.go:9
>> c
>> disass
>> b runtime.newproc
>> c
>> b runtime.newproc1
>> c
>> disass
>> b runtime.runqput # 将新创建的 G 放入 runnext（最多只能放入一个值），如果 runnext 之前有老的 G，则会尝试将其放入 local runnable queue（长度为 256 的数组） 的尾部，如果 local runnable queue 满了的话，会将 local runnable queue 的前一半以及新的 G 通过 runtime.runqputslow 将它们放入 global runnable queue（链表）的尾部
# 用户新创建的 G 一定会赋值给 runnext，局部性原理
```

消费端：

- runtime.shcedule：
  
  - 每从本地队列获取 60 次 G 会加锁通过 runtime.globalrunqget 从全局队列中获取头部的第一个 G；
    - 这个次数是通过 P 的 schedtick 来判断的，schedtick%61 == 0
    - 这里仅会获取全局队列头部的第一个 G，不会多获取
  - 如果当前不需要从全局队列获取 G 或者从全局队列没有获取到，会通过 runtime.runqget 从本地队列获取（优先从 runnext 获取，然后再是 local run queue）
  - 如果本地队列没获取到，会执行 runtime.findrunnable：
    - top 流程：
    - runtime.runqget 再次从本地队列获取
    - 没获取到，runtime.globalrunqget 加锁从全局队列获取
      - runtime.globalrunqget 会从全局队列拿 total/gomaxprocs + 1 个 G，但不能超过 128（local runnable queue 的一半），头部的 G 用于接下来执行，其他的会放入 local runnable queue
    - 没获取到，runtime.netpoll 从网络中获取（会是一个列表）
      - 如果获取到的话会执行并将其加锁放入全局队列中
    - 没获取到，runtime.runqsteal 从其他的 P 偷 local run queue 尾部的一半 g 回来放入本地队列
      - 执行的是偷到的 G 的最后一个
    - 没获取到，stop 流程：
    - 会垂死挣扎一下，从全局、本地、网络再检查一遍是否能获取到 g 不能的话进入休眠状态

- runtime.execute：执行 runtime.schedule 获取到的 G

- runtime.gogo：会把拿到的 G 的现场恢复出来，从 pc 寄存器开始继续执行

- runtime.goexit：缓存相关的 G 结构体资源，回到 runtime.schedule 继续执行

注意：每次操作全局队列都是加锁进行的，本地队列则是无锁队列，只需要使用 atomic 操作就可以保证并发安全，而不需要加锁。

# 处理阻塞

可以被 runtime 接管处理的阻塞是通过 **gopark/goparkunlock 挂起**和 **goready 恢复**的，所以只要找到 runtime.gopark 的调用方就可以知道在哪些地方会被 runtime 接管。

## runtime 处理

以下的阻塞会被 runtime 处理掉，而不会阻塞住调度循环，而是把 goroutine 挂起（让 g 先进某个数据结构—— sudog 或 g，待 ready 后再继续执行），此时，线程会进入 schedule，继续消费队列，执行其他 g，而不会让这个 g 一直占用该线程

- chan 阻塞（sudog）

```go
// channel send:
var send = make(chan int)
send <- 1
// channel recv:
var recv = make(chan int)
<- recv
```

- 睡眠（g）

```go
time.Sleep(time.Hour)
```

- select 阻塞（sudog）

```go
var (
    ch1 = make(chan int)
    ch2 = make(chan int)
)
// no case ready, block
select {
    case <- ch1:
        println("ch1 ready")
    case <- ch2:
    println("ch2 ready")
}
```

- 加锁（sudog）

```go
var lock sync.RWMutex
lock.Lock()
```

- 网络阻塞（g）

```go
// 1. net read
var c net.Conn
var buf = make([]byte, 1024)
// data not ready, block here
n, err := c.Read(buf)

// 2. net write
var c net.Conn
var buf = []byte("hello")
// send buffer full
```

为什么有的用 sudog 有的用 g？

一个 g 可能会被包装成多个 sudog，如 select 语句时。

## sysmon 处理

无法被 runtime 拦截的阻塞有：执行 c 代码（cgo）、阻塞在 syscall 上，这两种情况必须占用线程。

sysmon(system monitor) 是一个高优先级的后台线程，其 goroutine 在专有线程中执行，不需要绑定 P 就可以执行，主要的任务：

- checkdead：检查当前所有线程是否都被阻塞住了，如果被阻塞住了会崩溃提示
- netpoll：会在 sysmon 中执行一次
- retake：
  - 如果是 syscall 卡了很久，那就把 P 从 M 上剥离（handoffp）
  - 1.14 版本开始，如果是用户 G 运行很久了，那么发信号 SIGURG 抢占，之前的版本必须要主动让出才行

# 常见问题

## 输出顺序？

使用版本：**Go 1.13 及之前**

```go
// 输出顺序为 9 0 1 2 3 4 5 6 7 8
func main() {
    // 限制全局只有一个 P，这样当前线程创建的全部 G 都会进入同一个本地队列
    runtime.GOMAXPROCS(1)
    // i = 9 是最后创建的 G，在老版本（1.13及之前）里面一定会放到 runnext 中
    // 执行时，会先拿出 runnext 里面的 G，所以先输出的是 9
    // 然后 0~8 的 G 都是放在 local run queue 中，所以会顺序输出 0~8
    for i := 0; i < 10; i++ {
        i := i
        go func() {
            fmt.Println(i)
        }()
    }
    var ch = make(chan int)
    <-ch
}
```

```go
// 输出顺序为 0 1 2 3 4 5 6 7 8 9
func main() {
    runtime.GOMAXPROCS(1)
    for i := 0; i < 10; i++ {
        i := i
        go func() {
            fmt.Println(i)
        }()
    }
    // 老版本（1.13及之前）time.Sleep 也会创建一个 G，该 G 最后创建会被放到 runnext
    // 从而将 9 的 G 也放到了 local run queue 中，导致 0~9 的 G 都在 local run queue，所以会顺序输出 0~9
    time.Sleep(time.Hour)
}
```

# 数据结构

均在 runtime 包下。版本为 go 1.14

## chan

`ch := make(chan int, 3)`：调用 runtime.makechan 生成 hchan 的数据结构：

```go
type hchan struct {
    qcount   uint  // 底层 buffer 中当前一共有多少个数据
    dataqsiz uint  // size of the circular queue. 这里的值为 bufffer 的长度，即 3
    buf      unsafe.Pointer // 该指针指向了一个**循环数组**，数组长度为 chan buffer 的长度，即 3
    elemsize uint16
    closed   uint32
    elemtype *_type // element type
    sendx    uint   // send index，记录下一个可存放数据的索引
    recvx    uint   // receive index，记录读取 chan 的元素的数组索引
    recvq    waitq  // list of recv waiters，接收者的 goroutine 队列，链表的结构
    sendq    waitq  // list of send waiters，发送者的 goroutine 队列，链表的结构
    lock mutex
}
```

每次向 chan 中发送数据 runtime.chansend，会将数据放入 buffer 的循环数组中，sendx 记录下一个可存放数据的索引，当 buffer 满了以后，sendx 会变为 0，如果继续向 chan 发送数据时，qcount == dataqsize 代表 chan 的 buffer 满了，此时需要阻塞等待，就将 goroutine 及上下文保存成一个 sudog 的结构，将其挂到 sendq 上。接收数据类似。

chan 在发送、接受、关闭过程中都会有加锁操作的，所以是并发安全的。

- sender 挂起（runtime.gopark），一定是由 receiver(或 close)唤醒（runtime.goready）
- receiver 挂起（runtime.gopark），一定是由 sender(或 close)唤醒（runtime.goready）

## timer

定时器的实现方式一般有两种：小顶堆、时间轮算法（time wheel）。

Go 语言中 timer 是使用**小顶四叉堆**实现的。

在代码中执行 time.Sleep 或者 time.After 之类的，其本质都是创建了一个 timer，这个 timer 会被加到 runtime 维护的四叉堆中，堆的维护以 timer 的触发时间为准，距离当前时间最近触发的 timer 放在堆顶

- 早期实现，全局只有一个四叉堆：针对这个四叉堆会有一个专门的 G（timerproc） 来检查堆顶元素是否到期，到期就会触发，并会逐渐调整堆将所有需要触发的 timer 都触发完，再休眠
  - 用来休眠就近时间的方法依赖 futex timeout 机制
  - 缺点：全局大锁，并发量高时会影响整体吞吐量
- timers 数组最多 64 个四叉堆（降低锁粒度，相当于是分片锁）：每个 P 有自己的四叉堆，按照 P 的编号 hash 后从指定索引取四叉堆，8 个 P 的话，timers 就只有 8 个元素，每个元素是一个四叉堆，每个堆有自己的锁，与早期的四叉堆是完全一样的，也会各自有专门的 G（timerproc）来检查
  - 用来休眠就近时间的方法依赖 futex timeout 机制
  - 缺点：CPU 密集型任务会导致 timer 唤醒延迟
- 1.14 之后，去掉了 timers 数组，四叉堆直接和 P 绑定，去掉了之前四叉堆专门的 G（timerproc），而是在 runtime.schedule 中使用 runtime.checkTimers 来检查，每个 P 的四叉堆还是存在锁的，在 runtime 调度的 work-stealing 过程中，也会从其他 P 中偷 timer 处理。
  - 在每次 runtime.schedule 调度时都检查运行到期的定时器
  - runtime.sysmon 中会为 timer 未被触发（timeSleepUntil）兜底，启动新线程
  - 使用 netpoll 的 epoll wait 来做就近时间的休眠等待

[时间轮在 Kafka 的实践](https://www.infoq.cn/article/erdajpj5epir65iczxzi)

## map

在哈希表中查找某个键对应的元素时，先用哈希函数将键值转为哈希值（一个无符号整数），根据哈希值（的低几位）定位到一个哈希桶，再在哈希桶中找这个键值，由于键-元素时一起存储的，找到了键也就找到了元素。**会有两次比较的过程：比较哈希值从而找到哈希桶，比较键值从而找到键-元素对**。

- 高位哈希值：用来确定当前的 bucket（桶）有没有所存储的数据的。
- 低位哈希值：用来确定，当前的数据存在了哪个 bucket（桶）

```go
// map 的底层结构
type hmap struct {
    count     int // 元素的个数，len(map) 的返回值
    flags     uint8
    B         uint8  // 桶(buckets)数组的长度为 2^B 个，map 共计存放 loadFactor * 2^B 个元素
    noverflow uint16 // approximate number of overflow buckets; see incrnoverflow for details
    hash0     uint32 // hash seed

    buckets    unsafe.Pointer // 指向一个 2^B 大小的桶数组. may be nil if count==0.
    oldbuckets unsafe.Pointer // previous bucket array of half the size, non-nil only when growing
    nevacuate  uintptr        // 标识扩容进度，小于该地址的 buckets 已经完成迁移

    extra *mapextra // optional fields
}

// buckets 字段是地址，最终会指向这个结构体
type bmap struct {
    tophash [bucketCnt]uint8 // tophash 是 hash 值的高 8 位
}

// mapextra holds fields that are not present on all maps.
type mapextra struct {
    // If both key and elem do not contain pointers and are inline, then we mark bucket
    // type as containing no pointers. This avoids scanning such maps.
    // However, bmap.overflow is a pointer. In order to keep overflow buckets
    // alive, we store pointers to all overflow buckets in hmap.extra.overflow and hmap.extra.oldoverflow.
    // overflow and oldoverflow are only used if key and elem do not contain pointers.
    // overflow contains overflow buckets for hmap.buckets.
    // oldoverflow contains overflow buckets for hmap.oldbuckets.
    // The indirection allows to store a pointer to the slice in hiter.
    overflow    *[]*bmap
    oldoverflow *[]*bmap

    nextOverflow *bmap // 指向空闲的 overflow bucket 的指针
}
```

每个桶最多只能存放 8 个元素！

- 元素访问，可以看 runtime.mapaccess1_fast64，还有很多其他变种的底层函数

对 key 进行 hash 计算，然后使用 hash 值的 low bits 和 高 8 位找到对应的位置：

假设 B = 5，则 bucketMask = 11111，根据 hash值 和 bucketMask 按位与操作后的十进制值（实际上就是 low bits 的十进制数，不过 low bits 的长度与 bucketMask 也就是 B 相关）来决定去桶数组的哪个桶找；

找到桶以后，再根据 hash 值的高 8 位对应的十进制值从 bmap 结构的 tophash 中找到 key-value 对所处的位置，然后比较 key，返回对应的 value。

所以会比较两次，先根据 key 的 hash 比较及运算找到桶和桶内的具体位置，然后再比较 key 的值。

- 元素赋值，可以看 runtime.mapassign_fast64，还有很多其他变种的底层函数
- 元素删除，可以看 runtime.mapdelete_fast64，还有很多其他变种的底层函数
- map 缩容，Go 的 map 是不会缩容的！
- map 扩容，在 mapassign 的过程中会触发扩容，触发的条件有以下两种：

1、load factor 过大，即当前元素个数 >= 桶个数*6.5，这时就说明大部分桶可能都快满了，如果插入新元素，大概率要挂在 overflow 的桶上

解决方法：将 B 加一，进而 hmap 的桶数组容量会扩大为原来的两倍

2、overflow 的桶太多：noverflow 的数量 >= 2^15，或 noverflow 的数量 < 2^15 但 noverflow 的数量 > 桶的数量

上面两个情况都会被认为溢出桶过多，为啥会导致这种情况呢？是因为我们对 map 一边插入，一边删除，会导致其中很多桶出现空洞，这样使得 bucket 使用率不高，值存储得比较稀疏。在查找时效率会下降。

解决方法：移动桶的内容，使其元素紧密排列进而提高桶利用率，容量不变。

扩容过程中元素的搬运是渐进进行的。扩容过程中的以下 map 操作：

mapassign：将命中的 bucket 从 oldbuckets 顺⼿搬运到 buckets 中，顺便再多搬运⼀个 bucket

mapdelete：将命中的 bucket 从 oldbuckets 顺⼿搬运到 buckets 中，顺便再多搬运⼀个 bucket

mapaccess：优先在 oldbuckets 中找，如果命中，则说明这个 bucket 没有被搬运

搬运 bucket 时，会将该桶的 overflow 桶也⼀并搬完。

[深度解密 Go 语言之 map](https://mp.weixin.qq.com/s?__biz=MjM5MDUwNTQwMQ==&mid=2257483772&idx=1&sn=a6462bc41ec70edf5d60df37a6d4e966&scene=21#wechat_redirect)

[Map](https://github.com/cch123/golang-notes/blob/master/map.md)
