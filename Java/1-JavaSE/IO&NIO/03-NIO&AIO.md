JDK 1.4 中引入 NIO 框架(java.nio包)，提供了Channel、Selector、Buffer等抽象，可以构建多路复用的、同步非阻塞 IO 程序，同时提供了更接近操作系统底层的高性能数据操作方式。

JDK 1.7 中，NIO 有了改进，引入异步非阻塞 IO 方式，也叫 AIO(Asynchronous IO)。异步 IO 操作基于事件和回调机制，可理解为：应用操作直接返回，而不会阻塞，当后台处理完成，操作系统会通知相应线程进行后续工作。

|            IO             |              NIO              |
| :-----------------------: | :---------------------------: |
| 面向流（Stream Oriented） | 面向缓冲区（Buffer Oriented） |
|   阻塞IO（Blocking IO）   |  非阻塞IO（Non Blocking IO）  |
|          （无）           |      选择器（Selectors）      |

 Java NIO系统的核心在于：**通道(Channel)和缓冲区(Buffer)**。通道表示打开到 IO 设备(例如：文件、套接字)的连接。若需要使用 NIO 系统，需要获取用于连接 IO 设备的通道以及用于容纳数据的缓冲区。然后操作缓冲区，对数据进行处理。

简而言之，**Channel  负责传输， Buffer   负责存储**。



# 缓冲区Buffer

缓冲区是一个用于特定基本数据类型的容器，由`java.nio`包定义。所有的缓冲区都是 `Buffer`抽象类的子类。Java NIO 中的 Buffer 主要用于与 NIO 通道进行交互，数据是从通道读入缓冲区，从缓冲区写入通道中的。 

- Buffer 可以保存多个相同类型的数据。根据数据类型不同(boolean 除外) ，有以下 Buffer 常用子类：

  - `ByteBuffer`
  -  ``CharBuffer `、`ShortBuffer` 、`IntBuffer`、`LongBuffer`、`FloatBuffer`、`DoubleBuffer` 
  - `MappedByteBuffer`这个类是用来表示内存映射文件的
  - 都通过对应类的静态方法 `allocate()` 获取缓冲区，如`ByteBuffer buffer = ByteBuffer.allocate(1024)`



## 属性

缓冲区中的核心属性： 0 <= `mark` <= `position` <= `limit` <= `capacity` 

- **容量 (capacity)** ：表示 Buffer 最大数据容量，缓冲区容量不能为负，并且创建后不能更改。 
- **限制(limit)** ：第一个不应该读取或写入的数据的索引，即位于 limit 后的数据不可读写。缓冲区的限制不能为负，并且不能大于其容量。
- **位置(position)** ：下一个要读取或写入的数据的索引。缓冲区的位置不能为 负，并且不能大于其限制。
- **标记 (mark)** 与**重置 (reset)**：标记是一个索引，通过 Buffer 中的 `mark()`方法指定 Buffer 中一个特定的 position，之后可以通过调用 `reset()`方法恢复到这 个 position.。



## 使用

`Buffer`的**基本使用** :

- 1.写入数据到缓冲区

  - 方式一：从`Channel`写入`Buffer`。

    ```java
    int len = inChannel.read(buffer); 
    ```

  - 方式二：通过`Buffer`的`put()`方法写入`Buffer`。

    ```java
    buffer.put(存入的数据);//注意使用不同的Buffer实现类，传入的数据要是对应类型，如果是ByteBuffer，则写成 buffer.put(数据.getBytes());
    ```

- 2.调用`flip()`方法从写模式切换为读模式。会将position设为0，limit设为之前position的值。

- 3.读取缓冲区中的数据

  - 方式一：从`Buffer`读取数据到`Channel`。

    ```java
    int len = inChannel.write(buffer);
    ```

  - 方式二：使用`Buffer`的`get()`方法从`Buffer`读取数据。`get()`方法有多种不同版本

    ```java
    buffer.get();
    ```

  - `Buffer`的`rewind()`方法可以将 position 设为0，然后通过上面的方法重复读取`Buffer`中的数据。

- 4.调用`clear()`方法或`compact()`方法来清空缓存区：

  - `clear()`方法清空整个缓存区（但是数据还存在，只是处于“被遗忘”状态，无法知道有多少数据）；
  - `compact()`方法清除已读过的数据。
  - 一旦读完了所有的数据，就需要清空缓冲区，让它可以再次被写入。 

- `equals()`和`compareTo()`方法来比较两个`Buffer`

  - `equals()`方法：满足以下条件的两个`Buffer`相等
    - 有相同的类型（byte、char、int等）。
    - `Buffer`中剩余的byte、char等的个数相等。(剩余元素是从 position到limit之间的元素)
    - `Buffer`中所有剩余的byte、char等都相同。
  - `compareTo()`方法:满足以下条件，则一个`Buffer`小于另一个`Buffer`
    - 第一个不相等的元素小于另一个Buffer中对应的元素 。
    - 所有元素都相等，但第一个Buffer比另一个先耗尽(第一个Buffer的元素个数比另一个少)。

```java
@Test
public void testByteBuffer() {
  String s = "abcdef";
  //1.分配一个指定大小的缓冲区
  ByteBuffer buf = ByteBuffer.allocate(1024);

  System.out.println("***************allocate()****************");
  System.out.println(buf.position());    //0
  System.out.println(buf.capacity());    //1024
  System.out.println(buf.limit());  //1024

  //2.利用put()方法存入数据到缓冲区
  buf.put(s.getBytes());
  System.out.println("***************put()****************");
  System.out.println(buf.position());    //6
  System.out.println(buf.capacity());    //1024
  System.out.println(buf.limit());  //1024

  //3.flip()切换到读取数据模式
  buf.flip();
  System.out.println("***************flip()****************");
  System.out.println(buf.position());    //0
  System.out.println(buf.capacity());    //1024
  System.out.println(buf.limit());  //6

  //4.利用get()方法读取数据
  byte[] dst = new byte[buf.limit()];
  buf.get(dst);
  System.out.println("***************get()****************");
  System.out.println(buf.position()); // 6
  System.out.println(buf.capacity()); // 1024
  System.out.println(buf.limit());  //6
  System.out.println(new String(dst, 0, dst.length));

  //5.rewind() :可重复读数据
  buf.rewind();
  System.out.println("***************rewind()****************");
  System.out.println(buf.position()); // 0
  System.out.println(buf.capacity()); // 1024
  System.out.println(buf.limit()); // 6

  //5.clear() :清空缓冲区，但是数据还存在，只是处于“被遗忘”状态，无法知道有多少数据
  buf.clear();
  System.out.println("***************clear()****************");
  System.out.println(buf.position()); // 0
  System.out.println(buf.capacity()); // 1024
  System.out.println(buf.limit()); //1024
  System.out.println((char)buf.get()); //a
}
```

```java
@Test
public void testMark() {
  String s = "abcdef";
  ByteBuffer buf = ByteBuffer.allocate(1024);
  buf.put(s.getBytes());
  buf.flip();
  byte[] dst = new byte[buf.limit()];
  buf.get(dst,0,2);
  System.out.println(new String(dst));

  //mark()标记位置
  buf.mark();

  buf.get(dst, 2, 3);
  System.out.println(new String(dst,2,3));
  System.out.println(buf.position());    //5

  //reset()重置：恢复到mark()的位置
  buf.reset();
  System.out.println(buf.position());    //2

  //判断缓冲区中是否还有剩余的数据
  if(buf.hasRemaining()){
    //获取缓冲区中可以操作的数量
    System.out.println(buf.remaining());    //4
  }
}
```



## Direct Buffer

**直接缓冲区与非直接缓冲区** 

  - 非直接缓冲区：通过`allocate()`方法分配缓冲区，将缓冲区建立在 JVM 的内存中。
  - 直接缓冲区（只有`ByteBuffer`支持）：通过`allocateDirect()`方法分配缓冲区，将缓冲区建立在物理内存中，可以提高效率。
      - 部分内容见36讲的12-文件拷贝



# 通道Channel

通道是由 `java.nio.channels` 包定义的。`Channel` 表示 IO 源与目标打开的连接。`Channel` 类似于传统的“流”，但又有所不同， **既可以向通道写数据，也可以从通道读数据**，而流的读写通常是单向的；**通道可以异步地读写**；**通道只能和`Buffer`交互**。

- Java 为  `java.nio.channels.Channel`  接口提供的最主要实现类如下：
  - `FileChannel`：用于读取、写入、映射和操作文件的通道。不能切换成非阻塞模式
  - `DatagramChannel`：通过 UDP 读写网络中的数据。
  - `SocketChannel`：通过 TCP 读写网络中的数据。
  - `ServerSocketChannel`：可以监听新进来的 TCP 连接，对每一个新进来的连接都会创建一个 `SocketChannel`。 
- 获取通道： 
  - 方式1.Java针对支持通道的类提供了 `getChannel() `方法 
    - 支持通道的类： 
      - 本地IO：
        - `FileInputStream `/ `FileOutputStream`
        - `RandomAccessFile`
      - 网络IO：
        - `Socket`
        - `ServerSocket`
        - `DatagramSocket `
  - 方式2. JDK1.7中的NIO 针对各个通道提供了静态方法 `open()`
  - 方式3. JDK1.7中的NIO 的Files 工具类的 `newByteChannel() `



## 分散&聚集

- 分散 (Scatter) 和聚集 (Gather) 
  - 分散读取（Scattering Reads）是指从 `Channel` 中读取的数据“分散”到多个 `Buffer` 中。 
    - 注意：按照缓冲区的顺序，从 `Channel` 中读取的数据依次将 `Buffer` **填满**。
    - 要向下一个`Buffer`写入，前一个必须被填满。
  - 聚集写入（Gathering Writes）是指将多个` Buffer `中的数据“聚集”到 `Channel`。 
    - 注意：按照缓冲区的顺序，写入 **position 和 limit 之间的数据**到 `Channel `。

下面的例子以`FileChannel`为例：

```java
//分散和聚集
@Test
public void test4() throws IOException {
  RandomAccessFile raf1 = new RandomAccessFile("1.txt", "rw");

  //获取通道
  FileChannel channel1 = raf1.getChannel();

  //分配指定大小的缓冲区
  ByteBuffer buf1 = ByteBuffer.allocate(10);
  ByteBuffer buf2 = ByteBuffer.allocate(100);
  ByteBuffer buf3 = ByteBuffer.allocate(100);
  ByteBuffer buf4 = ByteBuffer.allocate(100);

  //分散读取。一个Buffer被写满后，Channel才向另一个Buffer写入
  ByteBuffer[] bufs = {buf1,buf2,buf3,buf4};
  channel1.read(bufs);

  for(ByteBuffer byteBuffer:bufs) {
    byteBuffer.flip();
  }
  //观察文件是否按顺序给不同缓存区写入数据
  System.out.println(new String(bufs[0].array(),0,bufs[0].limit()));
  System.out.println(new String(bufs[1].array(),0,bufs[1].limit()));
  System.out.println(new String(bufs[2].array(),0,bufs[2].limit()));
  System.out.println(new String(bufs[3].array(),0,bufs[3].limit()));

  // 聚集写入
  RandomAccessFile raf2 = new RandomAccessFile("2.txt", "rw");
  FileChannel channel2 = raf2.getChannel();

  channel2.write(bufs);
  channel2.close();
  channel1.close();
}
```



# 选择器

选择器用于使用单个线程处理多个通道。因此，它需要较少的线程来处理这些通道。线程之间的切换对于操作系统来说是昂贵的。 因此，为了提高系统效率选择器是有用的。

详见网络NIO部分的笔记



# StandardOpenOption类

枚举类`public enum StandardOpenOption`：`java.nio.file.StandardOpenOption`

使用详见获取通道 channel 的 open（）方法 。

- 定义了标准打开方法
  - `StandardOpenOption.WRITE`
  - `StandardOpenOption.READ`
  - `StandardOpenOption.CREATE_NEW ` 该文件不存在则创建，存在则报错
  - `StandardOpenOption.CREATE` 该文件不存在则创建，存在则覆盖 
  - `APPEND `
- `DELETE_ON_CLOSE`
- `SPARSE`
- `SYNC`
- `DSYNC` 

# SelectionKey类

  `SelectionKey`类表示 `SelectableChannel` 和`Selector`之间的注册关系。每次向选择器注册通道时就会选择一个事件（选择键）。选择键包含两个表示整数的操作集。操作集的每一位都表示该键的通道所支持的一类可选择操作。

- 常用方法：

  - `int interestOps()`：获取感兴趣事件集合

    ```java
    int interestSet = selectionKey.interestOps();
    //用“位与”操作interest 集合和给定的SelectionKey常量，可以确定某个确定的事件是否在interest 集合中。
    boolean isInterestedInAccept  = (interestSet & SelectionKey.OP_ACCEPT) == SelectionKey.OP_ACCEPT；
    boolean isInterestedInConnect = interestSet & SelectionKey.OP_CONNECT;
    boolean isInterestedInRead    = interestSet & SelectionKey.OP_READ;
    boolean isInterestedInWrite   = interestSet & SelectionKey.OP_WRITE;
    ```

  - `int readOps()`：获取通道已准备就绪的操作集合(可以想检测interest集合那样检测什么事件就绪，也可以使用下面四个方法)

  - `boolean isReadable()`：检测 `Channel` 中读事件是否准备就绪

  - `boolean isWritable()`：检测 `Channel `中写事件是否准备就绪

  - `boolean isConnectable()`：检测 `Channel` 中连接事件是否准备就绪

  - `boolean isAcceptable()`：检测 `Channel` 中接收事件是否准备就绪

  - `SelectableChannel channel()` ：获取注册通道

  - `Selector selector()`：返回选择器

  - `int select()`：监控所有注册的Channel，当它们中间有需要处理的 IO 操作时，该方法返回Channel的数量，并将对应的`SelectionKey`加入被选的`SelectionKey`集合中。阻塞到至少一个通道在所注册的时间上就绪。

  - `int select(long timeout)`：设置超时时长的`select()`操作。也会阻塞。

  - `int selectNow()`：执行一个立即返回的`select()`操作，该方法不会阻塞线程，不管什么通道就绪都立刻返回

    - 说明：select()方法返回的int值表示有多少通道已经就绪。亦即，自上次调用select()方法后有多少通道变成就绪状态。如果调用select()方法，因为有一个通道变成就绪状态，返回了1，若再次调用select()方法，如果另一个通道就绪了，它会再次返回1。如果对第一个就绪的channel没有做任何操作，现在就有两个就绪的通道，但在每次select()方法调用之间，只有一个通道就绪了。

  - `Set<SelectionKey> selectedKeys()`：所有的 `SelectionKey`集合，代表注册在该选择器上的Channel

  - `selectedKeys()` ：被选择的`SelectionKey`集合。返回此选择器已选的`SelectionKey`

  - `Selector wakeup()`：使一个还未返回的`select()`操作立即返回



# 字符集

```java
@Test
public void test5() throws CharacterCodingException {
  Map<String, Charset> map = Charset.availableCharsets();
  //       for(Entry<String, Charset> entry:map.entrySet()) {
  //           System.out.println(entry.getKey() + " : " + entry.getValue());
  //       }

  Charset cs1 = Charset.forName("GBK");
  //获取编码器
  CharsetEncoder ce = cs1.newEncoder();
  //获取解码器
  CharsetDecoder cd = cs1.newDecoder();

  CharBuffer cbuf = CharBuffer.allocate(1024);
  cbuf.put("fripSide!");
  cbuf.flip();

  //编码
  ByteBuffer buf = ce.encode(cbuf);
  //查看编码是否成功
  for(int i=0;i<9;i++) {
    System.out.println(buf.get());
  }

  //解码
  buf.flip();
  CharBuffer cbuf2 = cd.decode(buf);
  System.out.println(cbuf2.toString());

  System.out.println("+++++++++++用UFT-8解码++++++++++++++");
  buf.flip();
  Charset cs2 = Charset.forName("UTF-8");
  CharsetDecoder cd2 = cs2.newDecoder();
  CharBuffer c = cd2.decode(buf);
  System.out.println(c.toString());
}
```

