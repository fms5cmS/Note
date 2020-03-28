通常来说NIO中的所有IO都是从 Channel（通道） 开始的。

- 从通道进行数据读取 ：创建一个缓冲区，然后请求通道读取数据。
- 从通道进行数据写入 ：创建一个缓冲区，填充数据，并要求通道写入数据。

# FileChannel

文件IO主要是`FileChannel`类的使用。下面通过两个简单的例子来演示其如何使用

```java
//利用通道完成文件的复制（非直接缓冲区）getChannel() 方法
public static void copyFileByChannel(String source, String dest) {
  //1.获取通道
  try(FileChannel in = new FileInputStream(new File(source)).getChannel();
      FileChannel out = new FileOutputStream(new File(dest)).getChannel()){
    //2.分配缓冲区
    ByteBuffer buffer = ByteBuffer.allocate(1024);
    //3.将通道中的数据存入缓冲区
    while (in.read(buffer) != -1) {
      //切换读取数据模式
      buffer.flip();
      //4.将缓冲区的数据写入通道
      out.write(buffer);
      //清空缓冲区
      buffer.clear();
    }
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```

```java
//使用直接缓冲区完成文件的复制（内存映射文件）open()
public static void copyFileByDirectCache(String source, String dest) {
  try (FileChannel in = FileChannel.open(Paths.get(source), StandardOpenOption.READ);
       //CREATE_NEW 该文件不存在则创建，存在则报错；CREATE 该文件不存在则创建，存在则覆盖
       //这里既有Write也有Read是因为下面的outMappedBuf支持的模式只能是Read_Write
       FileChannel out = FileChannel.open(Paths.get(dest), StandardOpenOption.WRITE, StandardOpenOption.READ, StandardOpenOption.CREATE_NEW)) {

    //内存映射文件
    MappedByteBuffer inMappedBuf = in.map(FileChannel.MapMode.READ_ONLY, 0, in.size());
    MappedByteBuffer outMappedBuf = out.map(FileChannel.MapMode.READ_WRITE, 0, in.size());
    //直接对缓冲区进行读写操作
    byte[] buffer = new byte[inMappedBuf.limit()];
    inMappedBuf.get(buffer);
    outMappedBuf.put(buffer);
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```

- 其他方法：
  - `position()`方法：获取`FileChannel`的当前位置；
  - `position(long pos)`方法：设置`FileChannel`的当前位置；
  - `size()`方法：返回该实例所关联文件的大小；
  - `truncate(long size)`方法：截取一个文件。截取文件时，文件将中指定长度后面的部分将被删除。 
  - `force(boolean metaData)`方法：将通道里尚未写入磁盘的数据强制写到磁盘上，出于性能方面的考虑，操作系统会将数据缓存在内存中，所以无法保证写入到`FileChannel`里的数据一定会即时写到磁盘上。要保证这一点，需要调用该方法。 参数指明是否同时将文件元数据（权限信息等）写到磁盘上。 



# 通道间数据传输

通道之间的数据传输(直接缓冲区)；如果两个通道中**有一个是`FileChannel`**，那么我们可以直接将数据从一个`Channel`传输到另外一个`Channel`中。 以下方法是`FileChannel`的。

- `transferFrom(ReadableByteChannel src, long position, long count)`
  - src ：源通道；position：从通道position开始传输数据（非负）；count：传输的最大字节数（非负）
    `	transferTo(long position, long count, WritableByteChannel target) `
  - target：目标通道。

```java
//通道之间的数据传输(直接缓冲区)
public static void copyFileByChannelFrom(String source, String dest) {
  try(FileChannel in = FileChannel.open(Paths.get(source),StandardOpenOption.READ);
      FileChannel out = FileChannel.open(Paths.get(dest),StandardOpenOption.WRITE,StandardOpenOption.READ,StandardOpenOption.CREATE_NEW)) {
    //in.transferTo(0, in.size(), out);
    out.transferFrom(in,0,in.size());
  } catch (IOException e) {
    e.printStackTrace();
  }
}
```

