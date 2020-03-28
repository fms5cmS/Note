原文链接：[《Introduction to Java Bytecode》](https://dzone.com/articles/introduction-to-java-bytecode)

一篇 Java 字节码入门的文章，图文并茂地讲了 Java 字节码的几个简单指令以及变量在栈中位置的变化。这里列出了里面出现的一些指令(详见字节码指令[官方文档](https://docs.oracle.com/javase/specs/jvms/se8/html/jvms-6.html))：

- `int`型变量的部分指令：以下指令都是`int`型变量的操作指令
  - `iconst_<i>`：向操作数栈中压入`int`型常量 i
  - `istore_<n>`：将操作数栈顶的`int`型 value 弹出并存储到当前栈帧的局部变量数组的索引 n 处
  - `iload_<n>`：获取局部变量数组索引 n 处的变量(必须是`int`型)，将其压入操作数栈中
  - `iadd`：从操作数栈的栈顶弹出两个`int`值，相加后结果压入操作数栈顶
  - `i2d`：将操作数栈顶的`int`值扩展长度，转为`double`类型。`d2i`与`i2d`是相反的过程

- 方法调用：
  - `invokestatic`：调用类的静态方法，后面会跟一个当前类的运行时常量池中某个类或接口的静态方法的符号引用

    - 不太理解符号引用和直接引用的区别，只知道在类加载的解析阶段，JVM会将常量池内的符号引用替换为直接引用。

  - `invokespecial`：调用实例方法；对父类方法、私有方法、实例初始化方法调用的特殊处理。这里是查看的官方文档，可能有理解不到位的地方，所以下面给出了原文内容：

    >Invoke instance method; special handling for superclass, private, and instance initialization method invocations

    

    - 注意：使用构造函数创建对象实例时，会通过`dup`指令复制一份对象的引用并压入操作数栈，所以创建实例时，在操作数栈中会有两个相同的值指向同一个实例化的对象，其中复制出来的值(用于找到对象)会和构造函数的参数一起被“消费”从而创建实例，另一个则用于赋值给局部变量，便于用户调用，如果没有赋值操作的话则直接从栈中弹出。

  - `invokevirtual`：调用实例方法(不能是初始化方法)，而且可以根据类的实际类型来调用合适的方法。

    ```java
    class Father{
        public void say() {
            System.out.println("I'm father");
        }
    }
    public class Son extends Father{
        @Override
        public void say() {
            //查看该方法的字节码可以发现，调用父类方法用的是 invokespecial 指令
            super.say();
            System.out.println("I'm son");
        }
    
        public static void main(String[] args) {
    		//无论用父类接收还是子类接收，调用 say 方法的字节码都是 invokevirtual
            final Son sonn = new Son(); //初始化时用的是 invokespecial 指令
            //invokevirtual #7         // Method say:()V
            son.say();
        }
    }
    ```

  - `invokeinterface`：调用接口方法。在文章中并没有出现，这里是为了对比`invokevirtual`列出的

    ```java
    interface Shape{
        void area();
    }
    public class Square implements Shape {
        int a,b;
        public Square(int a, int b) {
            this.a = a;
            this.b = b;
        }
    
        @Override
        public void area() {
            System.out.println(a*b);
        }
    
        public static void main(String[] args) {
            //使用实际类型接收，调用 area 方法的字节码：
            // invokevirtual #8        // Method area:()V
            Square square = new Square(2, 3)
            square.area();
            //而如果使用接口 Shape 来接收，调用 area 方法的字节码：
            // invokeinterface #8,  1  // InterfaceMethod feature/Shape.area:()V
            Shape shape = new Square(2, 3)
            shape.area();   
        }
    }
    ```

  - 还有一个调用方法的指令`invokedynamic`，是用于调用动态方法，由于自己对底层的一些知识了解太少，所以看的有点不理解，这里没有说明，会等到以后再分享。

