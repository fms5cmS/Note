# 5.文件上传

## 5.1基础

- 进行文件上传时，表单的要求：

  - 请求方式必须为 POST。
  - 使用 file 的表单域，浏览器在解析表单时，会自动生成一个输入框和一个按钮，输入框可供用户填写本地文件的文件名和路径名，按钮可以让浏览器打开一个文件选择框供用户选择文件
  - 请求编码方式为 multipart/form-data

  ```jsp
      <!--由于上传文件是直接将文件附在请求后面，所以必须使用POST方式-->
      <form action="UploadServlet" method="post" enctype="multipart/form-data">
          File:<input type="file" name="file"/>
          <input type="submit" value="Submit"/>
      </form>
  ```

- 表单的 enctype 属性：

  - 用于指定将数据发送到服务器时浏览器使用的编码类型
  - 取值：
    - application/x-www-form-urlencoded：默认值。使用有限的字符集，当用了非字母和数字时，必用”%HH”代替(H代表十六进制数字)。对于大容量的二进制数据或包含非ASCII字符的文本来说，这种编码不能满足要求。
    - multipart/form-data：表单以二进制传输数据



服务端：

- 由于请求的编码方式是multipart/form-data，以二进制的方式来提交请求信息，所以无法再以`request.getParatemer()`等方式获取请求信息；
- 可以使用输入流的方式来获取. 但不建议这样做；
- 具体使用 commons-fileupload 组件来完成文件的上传操作！



## 5.2 commons-fileupload

该组件是 Apache 开源代码组织用来处理表单文件上传的一个子项目，该组件性能优异，可以支持任意大小的文件的上传。该包依赖于 commons-io 包。使用详见[官网](http://commons.apache.org/proper/commons-fileupload/using.html)。

```xml
<dependency>
    <groupId>commons-fileupload</groupId>
    <artifactId>commons-fileupload</artifactId>
    <version>1.3.3</version>
</dependency>
```

- 该组件可以解析请求，把所有的请求信息都解析为`FileItem`对象, 无论是一个一般的文本域还是一个文件域；

- 该组件会得到一个`FileItem`对象组成的`List`；

  - 简单方式：

  ```java
  // Create a factory for disk-based file items
  DiskFileItemFactory factory = new DiskFileItemFactory();
  
  // Create a new file upload handler
  ServletFileUpload upload = new ServletFileUpload(factory);
  
  // Parse the request
  List<FileItem> items = upload.parseRequest(request);
  ```

  - 复杂方式：可以为文件的上传加入一些限制条件和其他的属性

  ```java
  DiskFileItemFactory factory = new DiskFileItemFactory();
  
  //设置内存中最多可以存放的上传文件的大小, 若超出则把文件写到一个临时文件夹中. 以 byte 为单位
  factory.setSizeThreshold(yourMaxMemorySize);
  //设置那个临时文件夹（该文件夹必须已经存在）
  factory.setRepository(yourTempDirectory);
  
  ServletFileUpload upload = new ServletFileUpload(factory);
  
  //设置上传文件的总的大小. 也可以设置单个文件的大小
  upload.setSizeMax(yourMaxRequestSize);
  
  List<FileItem> items = upload.parseRequest(request);
  ```

- 可以调用`FileItem.isFormField()`方法来判断是一个 表单域 或不是表单域(则是一个文件域)

  ```java
  if (item.isFormField()) {  //表单域
      String name = item.getFieldName();
      String value = item.getString();
      //...
  } else { //文件域
      String fieldName = item.getFieldName();
      String fileName = item.getName();
      String contentType = item.getContentType();
      boolean isInMemory = item.isInMemory();
      long sizeInBytes = item.getSize();
  
      InputStream uploadedStream = item.getInputStream();
      //...
      uploadedStream.close();
  }
  ```

使用测试：

```java
public class UploadServlet extends HttpServlet {
    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        //1.得到FileItems的集合
        DiskFileItemFactory factory = new DiskFileItemFactory();

        //设置内存中最多可以存放的上传文件的大小, 若超出则把文件写到一个临时文件夹中. 以 byte 为单位
        factory.setSizeThreshold(1024 * 5);
        //设置那个临时文件夹
        factory.setRepository(new File("e:/files"));

        ServletFileUpload upload = new ServletFileUpload(factory);

        //设置上传文件的总的大小. 也可以设置单个文件的大小
        upload.setSizeMax(1024 * 1024 * 5);

        try {
            List<FileItem> items = upload.parseRequest(req);
            //2.遍历上面得到的集合,若是一个表单域则打印信息
            // 若是一个文件域，则把文件保存到e:\\files 目录下
            for (FileItem item:items) {
                if(item.isFormField()){
                    String fieldName = item.getFieldName();
                    String value = item.getString();
                    System.out.println(fieldName + " : " + value);
                } else {
                    String fieldName = item.getFieldName();
                    String fileName = item.getName();
                    String contentType = item.getContentType();
                    long sizeInBytes = item.getSize();
                    resp.getWriter().println("fieldName:"+fieldName+"\nfileName:"+fileName);
              	 resp.getWriter().println("fileType:"+contentType+";size:"+sizeInBytes/1024+"k");

                    //下面是在将上传的文件拷贝到目录下面
                    InputStream in = item.getInputStream();
                    byte[] buffer = new byte[1024];
                    int len = 0;

                    fileName = "e:\\files\\" + fileName;
                    System.out.println(fileName);

                    OutputStream out = new FileOutputStream(fileName);

                    while ((len = in.read(buffer)) != -1) {
                        out.write(buffer, 0, len);
                    }
                    out.close();
                    in.close();
                }
            }
        } catch (FileUploadException e) {
            e.printStackTrace();
        }
    }
}
```

也可查看[JSP自学手册](https://edu.aliyun.com/lesson_503_5655?spm=5176.10731542.0.0.6gHhHl#_5655)中的讲解



## 5.3 文件下载

- 静态下载

  ```jsp
  <a href="xyz.txt">download xyz.txt</a><!--下载文件：直接链接到服务器上的文件-->
  ```

- 情景：在一些网络系统中，需要隐藏下载文件的真实地址；或下载的文件需要动态的确认后再传送给客户端

- 方案：利用程序编码实现下载

  - 可以增加安全访问控制，只对经过授权认证的用户提供下载
  - 可以从任意位置提供下载的数据

- 利用程序实现下载需要实现两个响应头

  - **Web 服务器需要告诉浏览器其所输出的内容的类型**不是普通的文本文件或 HTML 文件，而**是一个要保存到本地的下载文件**。设置`Content-Type=application/x-msdownload`

    ```java
    response.setContentType("application/x-msdownload"); 
    ```

  - **Web 服务器希望**浏览器不直接处理相应的实体内容，而是**由用户选择将相应的实体内容保存到一个文件中**，这需要设置`Content-Disposition`。该响应头指定了接收程序处理数据内容的方式，在HTTP应用中只有 attachment是标准方式，attachment表示要求用户干预。在attachment后面还可以指定filename参数，该参数是服务器建议浏览器将实体内容保存到文件中的文件名称。在设置`Content-Dispostion`之前一定要指定`Content-Type`

    ```java
    response.setHeader("Content-Disposition", "attachment;filename=abc.txt");
    ```



设置完上述响应头后，具体的文件可以调用`response.getOutputStream`的方式, 以 IO 流的方式发送给客户端。

```java
OutputStream out = response.getOutputStream();
String pptFileName = "C:/Users/Think Pad/Desktop/文件.pptx";

InputStream in = new FileInputStream(pptFileName);

byte [] buffer = new byte[1024];
int len = 0;

while((len = in.read(buffer)) != -1){
	out.write(buffer, 0, len);
}

```

