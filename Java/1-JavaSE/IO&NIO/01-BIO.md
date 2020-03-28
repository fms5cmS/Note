BIO(Blocking I/O) 同步阻塞I/O模式，数据的读取写入必须阻塞在一个线程内等待其完成。即传统的`java.io`包，是基于流模型实现，提供了 File 抽象、输入输出了等 IO 功能。

# 认识

```java
byte[] bytes = str.getBytes(encoding);     // 编码
String str = new String(bytes, encoding)； // 解码
```

- 流的分类
  - 1.按流向分为：输入流与输出流 
  - 2.按传输数据类型分为
    - 字节流：二进制，可以处理一切文件
    - 字符流：文本文件，只能处理纯文本 
  - 3.按功能分为：
    - 节点流：节点是包裹源头的（字节流和字符流都属于节点流）
    - 处理流（必须在节点流之上）：可以对其他流处理，增强功能，提高性能 
- 文件读写字节流字符流
  - `FileInputStream`
  - `FileOutputStream`
  - ` FileReader`
  - `FileWriter`
- 带缓存的读写字节流字符流
  - `BufferedReader`
  - `BufferedWriter`
  - `BufferedInputStream`
  - `BufferedOutputStream`
- 专门处理数据的字节流字符流
  - `DataInputStream`
  - `DataOutputStream`
- 读取对象的字节流字符流
  - `ObjectInputStream`
  - `ObjectOutputStream`
- 转换流（字节流转换成字符流）
  - `InputStreamReader`
  - `OutputStreamWriter`
  - `PrintWriter`
  - `PrintStream`（标准的输出流，默认输出到控制台）



# 节点流

字节流、字符流：以字节流复制文件来说明使用步骤：

- 1.利用 File 类与目标文件建立联系

  ```java
  File fileIn = new File("e:/late in autumn.mp3");  // 读取的源文件
  File fileOut = new File("e:/favorite.mp3");  // 写入的目标文件
  ```

- 2.选择流（这里使用字节流）。**注意：通常会使用多态**

  - 这一步可能发生 `FileNotFoundException` 异常，注意处理。这里使用try-with-resource方式！

  ```java
  try(InputStream in =  new FileInputStream(fileIn); 
  	//true 表示以追加的形式来写入数据。如果不写或false，则以覆盖的形式写入。字符流用法相同
      OutputStream out = new FileOutputStream(fileOut,true)){
      //3.操作
  } 
  ```

- 3.操作：这一部分都是在try的`{}`内

  ```java
  //由于使用的是字节流，所以使用字节数组作为缓冲区来暂时保存读取到的数据。数组长度表示每次最多读到的数据长度。注意：如果选用的是字符流，就使用字符数组 chat[] 来保存
  byte[] buffer = new byte[1024]; 
  int len = 0;	//长度，便于后面的写入数据
  //利用输入流的 read(byte[] buffer) 方法将输入流读取到的数据暂存入字节数组中，返回值如果为 -1 表明读取到了末尾。使用该方法会产生 IOException 异常，注意处理
  while((len=in.read(buffer))!=-1){     
      //利用输出流的 write(byte b[], int off, int len) 方法将字节数组中起始索引为 0，数据长度为 len 的数据写入输入流连接到的目标文件中
      out.write(buffer,0,len);	
  }
  out.flush();   //清空缓冲区的数据，该方法要在流关闭前调用
  ```

  


拷贝文件

```java
public class ByteCopyFile {
  public static void copyFile(File src,File dest) throws IOException{
    if(!src.isFile()) {
      System.out.println("只能操作文件");
      throw new IOException();
    }
    //目标为已经存在的文件夹时，不能创建与文件同名的文件夹;如果是文件的话可以覆盖，文件夹不行
    if(dest.isDirectory()) {
      //dest = new File(dest,src.getName());
      throw new IOException("不能创建与文件同名的文件夹");
    }
    try (InputStream is = new FileInputStream(source);
         OutputStream os = new FileOutputStream(dest)){
      byte[] buffer = new byte[1024];
      int length;
      while ((length = is.read(buffer)) > 0) {
        //推荐使用这个write方法，由于最后读取的文件可能小于1024字节，所以就以实际读取的长度len为写入的长度，避免写入多余的空格
        os.write(buffer, 0, length);
      }
      os.flush();//强制刷出输出流中滞留的内容
      System.out.println("success！");
    }
  }
}
```

拷贝文件夹：

```java
/**
 * 文件夹的拷贝
 * 1.文件 复制 CopyFile中的copyFile()
 * 2.文件夹 创建 mikdirs()
 * 3.递归查找子级目录、文件
 */
public class ByteCopyDir {
  /**拷贝文件夹*/
  public static void copyDir(File src,File dest) {
    if(src.isDirectory()) {  //文件夹
      dest = new File(dest,src.getName());
      /** 父目录不能拷贝到子目录下 */
      if(dest.getAbsolutePath().contains(src.getAbsolutePath())) {
        System.out.println("父目录不能拷贝到子目录下");
        return;
      }
    }
    copyDirDetail(src,dest);
  }
  /**拷贝文件夹细节 */
  public static void copyDirDetail(File src,File dest) {
    if(src.isFile()) {  //文件
      try {
        ByteCopyFile.copyFile(src, dest);
      } catch (FileNotFoundException e) {
        e.printStackTrace();
      } catch (IOException e) {
        e.printStackTrace();
      }
    }else if(src.isDirectory()) {  //文件夹
      //确保目标文件夹存在
      dest.mkdirs();
      //获取下一级目录：文件
      for(File sub:src.listFiles()) {
        copyDirDetail(sub,new File(dest,sub.getName()));
      }
    }
  }
}
```

除了字节流、字符流，还有一种节点流，是字节数组流。

- 字节数组流
  - `ByteArrayInputStream` 使用方法同字节流
  - `ByteArrayOutputStream` 使用方法同字节流，新增了`toByteArray（）`方法，故无法使用多态



# 处理流

- 缓冲流：
  - 针对字节流的字节缓冲流
    - `BufferedInputStream `
    - `BufferedOutputStream `
  - 针对字符流的字符缓冲流 
    - `BufferedReader `  有新增方法：`readLine()`读取一行文本，所以无法使用多态
    - `BufferedWriter ` 有新增方法：`nextLine()`写入一个换行符，所以无法使用多态
- 转换流：字节流 转 字符流。常用来处理乱码问题
  - `OutputStreamWriter ` 编码
  - `InputStreamReader` 解码

```java
//为了防止出现乱码，在底层使用字节流，但是BufferedReader()不能直接使用字节流，故使用转换流对其处理
//可以指定解码字符集,要与编码字符集统一
BufferedReader br = new BufferedReader(
  new InputStreamReader(   //转换流
    new FileInputStream(   //字节流
      new File("e:/Test/Nine.java")),"utf-8")
);
```

- 处理基本数据类型+String类型  （可以保留数据+类型）

  - `DataInputStream `
  - `DataOutputStream `
  - 注意：输入流读取的顺序和输出流写入的顺序要一致，否则读取的数据会存在问题 
  - 如果出现 `EOFException表示 `没有读到相关内容 

- 处理引用类型 （可以保留数据+对象类型），也叫**序列化、反序列化**

  - 输入流（反序列化）：`ObjectInputStream`
  - 输出流（序列化）：`ObjectOutputStream `
  - 详见后面补充的序列化和反序列化部分

- PrintStream 打印流

  - 三个常量（默认指向为键盘和控制台）

    - 输入流：`System.in`键盘输入

    - 输出流：`System.out` 控制台输出信息；

      `System.err`控制台输出错误信息

    ```java
    //打印流指向的重定向: 利用System中的方法：
    setIn(InputStream in)、
    setOut（PrintStream out）、
    setErr(PrintStream err)、
    FileDescriptor.in 标准输入流：控制台输入    // 要借助FileOutputStream()
    FileDescriptor.out 标准输出流：控制台输出
    //eg:
    //1.控制台————>文件
    System.setOut(
        	new PrintStream(true，     //true表示自动刷新
        		new BufferedOutputStream(
                    new FileOutputStream(
                        new File("       "))));        
    System.out.println("test");//test就会写入到文件中
    //2.文件————>控制台
    System.setOut(
        new PrintStream(true，
        	new BufferedOutputStream(
    			new FileOutputStream(
    				FileDescriptor.in) ) ) );
    ```

# 关闭资源

- 借助自定义工具方法

  - 非泛型方法

    ```java
    public static void closeItem(Closeable... io) {
      for(Closeable temp:io) {
        if(null!=temp) {
          try {
            temp.close();
          } catch (IOException e) {
            e.printStackTrace();
          }
        }
      }
    }
    ```

  - 泛型方法

    ```java
    public static <T extends Closeable> void closeAll(T... io) {
      for(Closeable temp:io) {
        if(null!=temp) {
          try {
            temp.close();
          } catch (IOException e) {
            e.printStackTrace();
          }
        }
      }
    }
    ```

- JDK 1.7 新增特性 try-with-resource

  - 如果资源实现了`java.lang.AutoCloseable`（其中包括实现了`java.io.Closeable`接口的所有对象） ，就可以使用这种方式。 当 try 代码块结束时，自动释放资源，不需要显式的调用`close()`方法。 

  - 注意：

    - try 语句中声明的资源被隐式声明为 final ，资源的作用局限于带资源的 try 语句
    - 可以在一条 try 语句中管理多个资源，每个资源以`;`隔开即可。
    - 需要关闭的资源，必须实现了`AutoCloseable`接口或其子接口`Closeable`

  - 使用方法：

    ```java
    try(需要关闭的资源声明){
        //可能发生异常的语句
    }catch(异常类型 变量名){
        //异常的处理语句
    }finally{
        //一定执行的语句
    }
    ```

  - 示例：

    ```java
    try(InputStream is = new FileInputStream(new File("f:/src/path.txt"));
    	InputStream is2 = new FileInputStream(new File("f:/Animation.txt"))){
    	//操作：读取、写入
    }catch （Exception e）{
    	//如果操作中遇到异常，可以增加这一部分，也可以不写这一部分而在方法出声明异常
    }
    ```

    

# 序列化、反序列化

java 序列化是指把java对象转换为有序字节流，以便在网络上传输或保存在本地文件中的过程。反序列化则是客户端从文件或网络中获得序列化后的对象字节流后，重建对象的过程。

好处：实现了数据的持久化，通过序列化可以把数据永久保存在硬盘的文件中；实现了远程通信。

- 具体使用：

  - 输入流（反序列化）：`ObjectInputStream`   调用`readObject()`方法读取流中的对象
  - 输出流（序列化）：`ObjectOutputStream `    调用`writeObject()`方法输出可序列化对象

- 注意：

  - 先序列化后反序列化，反序列化顺序必须与序列化一致
  - 不是所有的对象都可以序列化 。**要想实现序列化，对象必须实现`java.io.Serializable`（一个标记接口，里面没有方法）或j`ava.io. Externalizable`接口**。如果序列化对象的属性也是一个对象，且要序列化，也必须实现序列化接口。
  - 不是所有的属性都需要序列化。**不想序列化的属性用`transient`标出，表明这是对象的临时数据 。一个静态变量不管是否被`transient`修饰，均不能被序列化** ，因为这是类的状态而不是对象的。
  - 序列化时，只对对象的状态进行保存，而不管对象的方法 
  - 如果一个对象的成员变量是一个对象，那么这个对象的数据成员也会被保存！这是能用序列化解决深拷贝的重要原因
  - 类的版本号：用于对象的序列化。具体用于读取对象时对比硬盘上对象的版本和程序中对象的版本是否一致，若不一致，读取失败，并抛出异常。 
  - 类的对象序列化后，类的属性有增加或者删除不会影响序列化，只是值会丢失。 
  - 如果父类序列化了，子类会继承父类的序列化，子类无需添加序列化接口。 
  - 如果父类没有序列化，子类序列化了，子类中的属性能正常序列化，但父类的属性会丢失，不能序列化。 
  - 序列化和反序列化的过程中会调用对象的构造器。 
  - 序列化是RMI（远程方法调用）过程中参数和返回值都必须实现的机制，而RMI是JavaEE的基础
  - 用Java序列化的二进制字节数据只能由Java反序列化，不能被其他语言反序列化。如果要进行前后端或者不同语言之间的交互一般需要将对象转变成Json/Xml通用格式的数据，再恢复原来的对象。 

- 案例：

  ```java
  //People类包含了 String name、Integer age、People son 属性，且实现了序列化接口
  @Test
  public void test() {
      //序列化到文件
      People son = new People("zs",10,null);
      People father = new People("zzk",30,son);
      try(ObjectOutputStream oos = new ObjectOutputStream(new FileOutputStream(new File("e:/a.txt")))){
          oos.writeObject(father);   //这里对整个对象进行了序列化，也可以只对属性序列化
      } catch (IOException e) {
          e.printStackTrace();
      } 
      //从文件中反序列化
      try(ObjectInputStream ois = new ObjectInputStream(new FileInputStream(new File("e:/a.txt")))) {
          People person = (People) ois.readObject();
          System.out.println(person);
      } catch (IOException e) {
          e.printStackTrace();
      } catch (ClassNotFoundException e) {
          e.printStackTrace();
      } 
  }
  ```

  ```java
  People{name='zzk', age=30, son=People{name='zs', age=10, son=null}} //输出
  ```

- 一个对象想要序列化，有以下三种方法：

  - 该对象仅实现了`Serializable` 接口，按照以下方式实现：
    - `ObjectOutputStream` 采用默认的序列化方式来对非`transient`的属性序列化
    - `ObjectInputStream` 采用默认的反序列化方式来对序列化后的属性反序列化
  - 该对象实现了 `Serializable` 接口，还定义了`readObject(ObjectInputStream in)`和`writeObject(ObjectOutputSteam out) ` 方法，按照以下方式实现：
    - `ObjectOutputStream` 调用该对象的`writeObject(ObjectOutputSteam out) `对非`transient`的属性序列化
    - `ObjectInputStream` 调用该对象的`readObject(ObjectInputStream in)`对序列化后的属性反序列化
  - 该对象实现了 `Externalnalizable` 接口，且必须实现`readExternal(ObjectInput in) `方法和`writeExternal(ObjectOutput out) `方法，则按照以下方式实现：
    - `ObjectOutputStream`调用该对象的`writeExternal(ObjectOutput out)`对非`transient`的属性序列化
    - `ObjectInputStream`调用该对象的`readExternal(ObjectInput in)`对序列化后的属性反序列化

