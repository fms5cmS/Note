# 1.类加载

## 1.1 类加载过程

类从被加载到虚拟机内存中开始，到卸载出内存为止，它的整个生命周期包括：**加载（Loading）、验证（Verification）、准备(Preparation)、解析(Resolution)、初始化(Initialization)、使用(Using)和卸载(Unloading)7个阶段**。其中准备、验证、解析3个部分统称为连接（Linking）。如图所示：

![类加载过程](../../../images/Reflection-load.png)

1. 在实际类加载过程中，JVM 会将所有的`.class`字节码文件中的**二进制数据**读入内存中，导入运行时数据区的**方法区**中。
2. 当一个类首次被**主动加载**或**被动加载**时，类加载器会对此类执行类加载的流程 – **加载**、**连接**（**验证**、**准备**、**解析**）、**初始化**。
3. 如果类加载成功，**堆内存**中会产生一个新的`Class`对象，`Class`对象封装了类在**方法区**内的**数据结构**。

注意：加载、验证、准备、初始化和卸载这5个阶段的顺序是确定的，类的加载过程必须按照这种顺序按部就班地开始，而解析阶段则不一定：它在某些情况下可以在初始化阶段之后再开始，这是为了支持Java语言的运行时绑定（也称为动态绑定或晚期绑定）。

加载阶段和连接阶段（Linking）的部分内容（如一部分字节码文件格式验证动作）是交叉进行的，加载阶段尚未完成，连接阶段可能已经开始，但这些夹在加载阶段之中进行的动作，仍然属于连接阶段的内容，这两个阶段的开始时间仍然保持着固定的先后顺序。 以下陈述的内容都以HotSpot为基准。 

- ==加载==：查找并加载类的二进制数据
  - 通过一个类的全限定名来获取定义此类的二进制字节流（并没有指明要从一个Class文件中获取，可以从其他渠道，譬如：网络、动态生成、数据库等）； 
  - 将这个字节流所代表的**静态存储结构**转化为**方法区**的**运行时数据结构**； 
  - 在内存中（没有规定是在 Java 堆）生成一个代表这个类的`java.lang.Class`对象，作为方法区这个类的各种数据的访问入口。
- ==验证==：确保 Class 文件的字节流中包含的信息符合当前虚拟机的要求，并且不会危害虚拟机自身的安全。验证阶段大致会完成4个阶段的检验动作： 

  - 文件格式验证：验证字节流是否符合 Class 文件格式的规范，且能被当前版本的 JVM 处理；
    - 如：是否以魔数`0xCAFEBABE`开头、主次版本号是否在当前虚拟机的处理范围之内 等。 
    - 这一阶段是基于二进制字节流进行的，只有通过该阶段验证后，字节流才会进入内存的方法区中进行存储，故后面三个验证阶段都是基于方法区的存储结构进行的，不会再直接操作字节流。
  - 元数据验证：对字节码描述的信息进行语义分析，以保证其描述的信息符合 Java 语言规范的要求；（**对元数据信息中的数据类型校验**）
    - 例如：这个类是否有父类（除了`java.lang.Object`之外，所有类都应有父类）
  - 字节码验证：通过数据流和控制流分析，确定程序语义是合法的、符合逻辑的；（**对类的方法体校验分析**）
  - 符号引用验证：确保解析动作能正确执行。 
  - 注意：验证阶段是非常重要的，但不是必须的，它对程序运行期没有影响，如果所运行的全部代码已经过反复验证，那么可以考虑采用`-Xverifynone`参数来关闭大部分的类验证措施，以缩短虚拟机类加载的时间。 
- ==准备==：正式为类变量分配内存并设置类变量初始值，这些变量所使用的内存都将在方法区中进行分配。

  - 进行内存分配的仅包括**类变量（被static修饰的变量）**，而不包括实例变量。
  - 实例变量将会在对象实例化时随着对象一起分配在堆中；
  - 这里所说的”初始值“通常情况下是数据类型的**默认零值**。
    - 如：`public static int value = 123`在准备阶段后，值为 0 ，赋值为 123 的`putstatic`指令是在程序被编译后，存放于类构造器`<clinit>()`方法中，即赋值为 123 的动作在初始化阶段执行。
  - 特殊：如果**类变量同时还是常量即被`final`修饰**，则在准备阶段就会被赋值
    - 如：`public static final int value = 123`会在准备阶段就初始化为指定值，即 `value`的值在准备阶段为 123 而不是 0。
- ==解析==：虚拟机将**常量池**内的**符号引用**替换为**直接引用**。

  - 主要针对**类或接口、字段、类方法、接口方法、方法类型、方法句柄和调用点限定符**7类符号引用进行。 
- ==初始化==：对**类静态变量**赋予正确的初始值 ，执行类构造器`<clinit>()`方法的过程。
  - 目标：
    - 实现对声明**类静态变量**时指定的初始值的初始化；
    - 实现对使用**静态代码块**设置的初始值的初始化。

  - `<clinit>()`方法是由编译器自动收集类中的所有类变量的赋值动作和静态语句块`static{}`中的语句合并产生的，编译器收集的顺序是由语句在源文件中出现的顺序所决定的，静态语句块只能访问到定义在静态语句块之前的变量，定义在它之后的变量，在前面的静态语句块可以赋值，但是不能访问 。 

    ```java
    public class Test{
        static{
            i = 0;   //给变量赋值可以正常编译通过
            System.out.println(i); //编译器提示“非法向前引用”
        }
        static int i = 1;
    }
    ```

  - 当初始化一个类的时候，如果发现其父类还没有进行过初始化、则需要先对其父类进行初始化。

  - 虚拟机会保证一个类的`<clinit>()`方法在多线程环境中被正确加锁和同步。 

  - `<clinit>()`方法并不是必须的，如果一个类中没有静态语句块，也没有对变量的赋值操作，则编译器可以部位该类生成`<clinit>()`方法。

  - 初始化的时机：
    - 创建类的实例(`new`关键字)；
    - `java.lang.reflect`包中的方法(如：`Class.forName(“xxx”)`)；
    - 对类的**静态变量**进行访问或赋值；
    - 访问调用类的**静态方法**；
    - 初始化一个类的**子类**，**父类**本身也会被初始化；
    - 作为程序的**启动入口**，包含`main`方法(如：`SpringBoot`入口类)。 



## 1.2 主/被动引用

- **类的主动引用**（一定会发生类的初始化） 在类加载阶段，会执行**加载**、**连接**和**初始化**操作。
  - 遇到`new`、`getstatic`、`putstatic`、`invokestatic`这四条字节码指令时；
    - 代码场景：使用`new`关键字实例化对象、读取或设置一个类的静态字段（被`final`修饰、已在编译器把结果放入常量池的静态字段除外）、调用类的静态方法时。
  - 使用`java.lang.reflect`包的方法对类进行反射调用；
  - 当虚拟机启动时，先初始化主类（main方法所在的类）；
  - 当初始化一个类，如果其父类没有被初始化，则先会初始化他的父类；
  - 当使用JDK 1.7的动态语言支持时，如果一个`java.lang.invoke.MethodHandle`实例最后的解析结果`REF_getStatic`、`REF_putStatic`、`REF_invokeStatic`的方法句柄，并且这个方法句柄所对应的类没有进行过初始化，则需要先触发其初始化。
- **类的被动引用**(不会发生类的初始化) 在类加载阶段， 只执行**加载**、**连接**操作，不触发**初始化**操作。 
    - 当访问一个静态域时，只有真正声明这个域的类才会被初始化。
      - 即 通过子类引用父类的静态变量，不会导致子类初始化，会初始化父类。
    - 通过数组定义来引用类而不赋值，不会触发此类的初始化：`A[] a = new A[10]` 
    - 引用常量不会触发此类的初始化（常量在编译阶段就存入调用类的常量池中了）。



## 1.3 类的加载顺序

1. 父类静态对象和静态代码块
2. 子类静态对象和静态代码块
3. 父类非静态对象和非静态代码块
4. 父类构造函数
5. 子类 非静态对象和非静态代码块
6. 子类构造函数 

注意：静态块按照声明顺序执行！

- 说明：
  - 静态初始化块，静态变量这两个是属于同一级别的，是按代码写得顺序执行的。
  - `main()`方法是静态的，因此JVM在执行`main()`方法时不创建`main()`方法所在的类的实例对象。
  - 如果`main()`方法所在类的内部也有其他静态初始化块 A ，且在`main()`方法前面，则先执行A，再执行`main()`方法。然后根据`main()`方法再加载类。 

# 2.类加载器

​	类加载器：将 class 文件字节码内容加载到内存中，并将这些静态数据转换成方法区中的运行时数据结构，在堆中生成一个代表这个类的`java.lang.Class`对象，作为方法区类数据的访问入口。 

## 2.1 层次结构

类加载器之间的父子关系一般通过组合（Composition）关系来实现，而不是通过继承（Inheritance）的关系实现。 

- 引导类加载器（bootstrap ClassLoader）（C++实现）
  - 在`Java`虚拟机启动后初始化的。
  - 负责加载 `ExtClassLoader`，并且将 `ExtClassLoader`的父加载器设置为 `Bootstrap Classloader`
  - `Bootstrap Classloader` 加载完 `ExtClassLoader` 后，就会加载 `AppClassLoader`，并且将 `AppClassLoader` 的父加载器指定为 `ExtClassLoader`。
  - 用来加载 Java 的核心库(`%JAVA_HOME%/jre/lib/rt.jar`,或`sun.boot.class.path`路径下的内容)，是用原生代码C++来实现的，并不继承自`java.lang.ClassLoader`。
  - 加载扩展类和应用程序类加载器。并指定他们的父类加载器。



下面的三个类加载器都继承`java.lang.ClassLoader`

- 扩展类加载器（extensions ClassLoader）（ JAVA编写）
  - 用来加载 Java 的扩展库(`%JAVA_HOME%/jre/lib/ext/*.jar`，或`java.ext.dirs`路径下的内容) 。
  - Java 虚拟机的实现会提供一个扩展库目录。该类加载器在此目录里面查找并加载 Java类。 
  - 由`sun.misc.Launcher$ExtClassLoader`实现。
- 应用程序类加载器（application class loader）（ JAVA编写）
  - 根据 Java 应用的类路径（classpath，`java.class.path` 路径下的内容）来加载 Java 类。一般来说，Java 应用的类都是由它来完成加载的。 
  - 由`sun.misc.Launcher$AppClassLoader`实现。

- 自定义类加载器 （ JAVA编写）
  - 开发人员可以通过继承 java.lang.ClassLoader类的方式实现自己的类加载器，以满足一些特殊的需求。



**类加载器的特点**

- **层级结构**：Java里的类装载器被组织成了有父子关系的层级结构。Bootstrap类装载器是所有装载器的父亲。
- **代理模式**： 基于层级结构，类的代理可以在装载器之间进行代理。当装载器装载一个类时，首先会检查它在父装载器中是否进行了装载。如果上层装载器已经装载了这个类，这个类会被直接使用。反之，类装载器会请求装载这个类
- **可见性限制**：一个子装载器可以查找父装载器中的类，但是一个父装载器不能查找子装载器里的类。
- **不允许卸载**：类装载器可以装载一个类但是不可以卸载它，不过可以删除当前的类装载器，然后创建一个新的类装载器装载。

 

 **类加载器的隔离问题**

​	每个**类装载器**都有一个自己的**命名空间**用来保存已装载的类。当一个类装载器装载一个类时，它会通过保存在命名空间里的**类全局限定名**进行搜索来检测这个类是否已经被加载了。

​       `JVM` 及 `Dalvik` 对类唯一的识别是 `ClassLoader id` + `PackageName` + `ClassName`，所以一个运行程序中是有可能存在两个**包名**和**类名**完全一致的类的。并且如果这两个**类**不是由一个 `ClassLoader` 加载，是无法将一个类的实例**强转**为另外一个类的，这就是 `ClassLoader` 隔离性。

​	为了解决类加载器的**隔离问题**，`JVM`引入了**双亲委托机制**。

 

## 2.2 双亲委托机制

**自底向上**检查类是否**已加载**；

**自顶向下**尝试**加载类**。 

- 双亲委托机制：
  - 代理模式的一种。
  - 某个特定的类加载器在接到加载类的请求时，首先将加载任务委托给父类加载器，依次追溯，直到最高的爷爷辈的，如果父类加载器可以完成类加载任务，将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实例对象；只有父类加载器无法完成此加载任务时，才自己去加载。 
  - 双亲委托机制是为了保证 Java 核心库的类型安全。因为String已经在启动时就被引导类加载器加载，所以用户自定义的ClassLoader永远也无法加载一个自己写的 String，除非你改变JDK中 ClassLoader 搜索类的默认算法。 
  - 这种机制就保证不会出现用户自己能定义`java.lang.Object`类的情况。 
  - 并不是所有的类加载器都采用双亲委托机制。 
  - tomcat服务器类加载器也使用代理模式，所不同的是它是首先尝试去加载某个类，如果找不到再代理给父类加载器。这与一般类加载器的顺序是相反的。



**JVM在判定两个class是否相同时，不仅要判断两个类名是否相同，而且要判断是否由同一个类加载器实例加载的。只有两者同时满足的情况下，JVM才认为这两个class是相同的。** 

可见性：子类加载器可以看到父类加载器加载的类，而反之则不行。

单一性：父加载器加载过的类不能被子加载器加载第二次。 



## 2.3 java.lang.ClassLoader 

`java.lang.ClassLoader`类的基本职责就是根据一个指定的类的名称，找到或者生成其对应的字节代码，然后从这些字节代码中定义出一个Java 类，即 `java.lang.Class`类的一个实例。同时还负责加载 Java 应用所需的资源，如图像文件和配置文件等。

- 相关方法：
  - `getParent()` 返回该类加载器的父类加载器。
  - ` loadClass(String name) `加载名称为 name的类，返回的结果是 `java.lang.Class`类的实例。 
    - 负责加载指定名字的类，首先会从已加载的类中去寻找，如果没有找到；从`parent ClassLoader[ExtClassLoader]`中加载；如果没有加载到，则从`Bootstrap ClassLoader`中尝试加载(`findBootstrapClassOrNull`方法), 如果还是加载失败，则自己加载。如果还不能加载，则抛出异常`ClassNotFoundException`。
    -  如果要改变类的加载顺序可以覆盖此方法； 
  - `findClass(String name)` 查找名称为 name的类，返回的结果是 `java.lang.Class`类的实例。 
  - `findLoadedClass(String name)` 查找名称为 name的已经被加载过的类，返回的结果是 `java.lang.Class`类的实例。 
  - `defineClass(String name, byte[] b, int off, int len) `把字节数组 b中的内容转换成 Java 类，返回的结果是`java.lang.Class`类的实例。这个方法被声明为 `final`的。 
  - `resolveClass(Class<?> c)` 链接指定的 Java 类。 
  - 注意：对于以上给出的方法，表示类名称的 name 参数的值是类的二进制名称。需要注意的是内部类的表示，如    `com.example.Sample$1和com.example.Sample$Inner`等表示方式。 

```java
public class ClassLoaderTest {
  public static void main(String[] args) throws ClassNotFoundException {
    //1.获取一个系统的类加载器
    ClassLoader classLoader = ClassLoader.getSystemClassLoader();
    System.out.println(classLoader);

    //2.获取系统类加载器的父类加载器
    classLoader = classLoader.getParent();
    System.out.println(classLoader);

    //3.获取扩展类加载器的父类加载器——引导类加载器（无法获取到的）
    classLoader = classLoader.getParent();
    System.out.println(classLoader);

    System.out.println(System.getProperty("java.class.path"));//当前项目的类路径有那些

    //4.测试当前类是由哪个类加载器进行加载
    classLoader = Class.forName("class_study.ClassLoaderTest").getClassLoader();
    System.out.println(classLoader);

    //5.测试 JDK 提供的 Object 类是由哪个类加载器加载的(引导加载器)
    classLoader = Class.forName("java.lang.Object").getClassLoader();
    System.out.println(classLoader);

    //6.关于类加载器的一个主要方法.
    //调用 getResourceAsStream() 获取路径下的文件对应的输入流
    InputStream in = null;
    //在静态方法中不能使用 this 关键字
    //       in = this.getClass().getClassLoader().getResourceAsStream("class_study/test.properties");
    ClassLoaderTest clt = new ClassLoaderTest();
    in = clt.getClass().getClassLoader().getResourceAsStream("class_study/test.properties");
    System.out.println(in);
  }
}
```



## 2.4 自定义类加载器

- 自定义类加载器的流程： 必须继承`java.lang.ClassLoade`
  1. 检查请求的类型是否已经被这个类装载器装载到命名空间中了，如果已经装载，直接返回；否则转入步骤2 
  2. 委派类加载请求给父类加载器（更准确的说应该是双亲类加载器，虚拟机中各种类加载器最终会呈现树状结构），如果父类加载器能够完成，则返回父类加载器加载的Class实例；否则转入步骤3 
  3. 调用本类加载器的`findClass（…）`方法，试图获取对应的字节码，如果获取的到，则调用`defineClass（…）`导入类型到方法区；如果获取不到对应的字节码或者其他原因失败，返回异常给`loadClass（…）`， `loadClass（…）`转抛异常，终止加载过程（注意：这里的异常种类不止一种）。 

# 3. 反射

反射机制： 指的是可以在运行时加载、探知、使用编译期间完全未知的类。 

程序在运行状态中，可以动态加载一个只有名称的类，对于任意一个已加载的类，都能知道这个类的所有属性和方法；对于任意一个对象，都能调用它的任意一个方法和属性。

加载完类之后，在堆内存中，就产生了一个 Class 类型的对象（一个类只有一个 Class 对象），这个对象就包含了完整的类的结构信息。我们可以通过这个对象看到类的结构。这个对象就像一面镜子，透过这个镜子看到类的结构，所以，我们形象的称之为：反射。 

 一个类只会被加载一次，同时在JVM中只形成一个对应的Class对象。 

只要类型相同，得到的Class对象就是同一个。 

对于数组，类型和维数的不同都会导致产生的Class对象不同，而长度则不会影响。

枚举被当作类，注解被当作接口。 



## 3.1 获取反射对象

- 得到Class 对象（三种方法） 

  - 直接通过类名 . class 的方法得到。该方法最为安全可靠，程序性能高。

    ```java
    Class clazz = int.class
    ```

  - 通过对象的 getClass() 方法获取,通常在不知道对象具体是什么类时用。

  - 通过全类名的方式获取。有可能抛出`ClassNotFoundException`异常。（用的较多！！） 

    ```java
    Class c = Class.forName(String className)
    ```



## 3.2 反射获取信息

```java
//获取类名
System.out.println(clazz.getName());   //获得包名.类名
System.out.println(clazz.getSimpleName()); //获得类名

//获得属性信息
Field[] fields = clazz.getFields();     //获得所有的公共属性
Field[] fields2 = clazz.getDeclaredFields(); //获得所有声明了的属性（含private）
Field f = clazz.getDeclaredField("name");   //获得指定的属性

//获得方法信息
Method[] methods = clazz.getMethods(); //获得所有公共方法
Method[] methods2 = clazz.getDeclaredMethods();//获得所有声明的方法

//获得指定的方法，需要传入方法的参数类型以区分重载
Method method = clazz.getMethod("getName", null);
Method m = clazz.getMethod("main",String[].class)       //获得main方法
Method method2 = clazz.getMethod("setName", String.class);

//获得构造器
Constructor[] constructors = clazz.getDeclaredConstructors();//获得所有构造方法

//获得指定构造方法，传入构造方法的参数
Constructor constructor = clazz.getConstructor(int.class,int.class,String.class);
```



## 3.3 反射操作方法、属性 

```java
//通过反射API调用构造方法来构造对象
User u = (User)clazz.newInstance(); //其实是调用了User的无参构造方法

//调用指定的构造方法来构造对象
Constructor<User> c = clazz.getConstructor(int.class,int.class,String.class);
User u2 = c.newInstance(1001,24,"zzk");                     //传入参数
System.out.println(u2.getName());

//通过反射API调用普通方法
User u3 = clazz.newInstance();
Method method = clazz.getDeclaredMethod("setName", String.class);
method.invoke(u3, "nanjo");             //u3.setName("nanjo")
method.invoke(null , (Object)new String[]{"aa","bb"})   //调用main方法。如果是调用static方法，第一个参数为null
//必须加类型转换，否则编译器会把数组拆分成两个参数，而不再是一个String数组参数
method.invoke(null , (Object)String[]{"s","d"} , (Object)String[]{"f","a"} )       //调用静态方法，有多个参数
    
//通过反射API操作属性
User u4 = clazz.newInstance();
Field f = clazz.getDeclaredField("name");
f.setAccessible(true); //这个属性不用做安全检查，可以直接访问。从而对private属性操作
f.set(u4, "fripSide"); //通过反射设置u4的name属性
 System.out.println(f.get(u4)); //通过反射得到u4的name值
```

