# final 数据

编译时常量：编译器可以将该常量代入任何可能用到它的计算式中，即可以在编译时执行计算式，减轻运行时的负担。**编译时常量必须以关键字`final`表示，且在进行定义时，必须对其进行赋值**。案例：

```java
byte b1 = 1,b2 = 2,b3,b6;
final byte b4 = 4,b5 = 6;
b6 = b4 + b5;                //10
b3 = (b1 + b2);                //b1、b2先提升为int再计算，导致编译出错           
//解释：被final修饰的变量是常量，b6在编译时就已经变成b6 = 10了。而b1和b2是byte类型，
//在计算时先提升为int类型再进行计算，所以编译无法通过，需要进行强制转型。
//笔记：当short，byte，char参加运算时，结果为int型，而不是与较高的类型相同。如果变量
//是byte，short，char类型，当对其赋予编译时期的常量，而该常量又没有超过变量的取值范围
//时，编译器就可以进行隐式的收缩转换。这种隐式的收缩转换是安全的，因为该收缩转换只适用
//于变量的赋值，而不适用于方法调用语句，即不适用于方法调用时的参数传递。
```

- 一个既是`static`又是`final`的数据只占一段不能改变的存储空间。

- 对于基本数据类型，`final`使其数值恒定不变；对于引用类型，`final`使其引用恒定不变。一旦引用被初始化指向一个对象，就无法再把它改为指向另一个对象，不过所指向的对象自身确实可以被修改的。

- 空白`final`

  - java 允许生成空白`final`，即在声明时没有初始化。
  - 无论什么情况，编译期都确保空白`final`在使用前必须被初始化。即必须在该常量声明处或构造器中用表达式对其赋值。

```java
import java.util.Random;

public class FinalData {
  private static Random rand = new Random(47);
  private String id;
  private final int a; //空白 final

  public FinalData(String id) {
    this.id = id;
    a = 10;  //必须在声明处或构造器中对空白final赋值，否则编译期报错。
  }

  //在编译期就已经知道数值的常量:
  private final int valueOne = 9;
  private static final int VALUE_TWO = 99;

  //典型公共常量：public说明可用于包外；static说明只有一份；final说明是一个常量:
  //注意：带恒定初始值（即编译期常量）的final static基本类型全用大写字母命名，且字与字间用下划线分隔
  public static final int VALUE_THREE = 39;

  //在运行时使用随机生成的数值来初始化常量，所以编译期并不知道数值！！！！:
  //i4在fd1和fd2中的值是唯一的
  private final int i4 = rand.nextInt(20);
  //由于static在装载时已被初始化，INT_5的值是不能通过创建第二个FinalData对象而改变
  static final int INT_5 = rand.nextInt(20);

  @Override
  public String toString() {
    return "FinalData{" +  "id='" + id + '\'' +  ", valueOne=" + valueOne +  ", i4=" + i4 +  ",INT_5=" + INT_5 + '}';
  }

  public static void main(String[] args) {
    FinalData fd1 = new FinalData("fd1");
    System.out.println(fd1);
    FinalData fd2 = new FinalData("fd2");
    System.out.println(fd1);
    System.out.println(fd2);
  }
}
```

```java
FinalData{id='fd1', valueOne=9, i4=15,INT_5=18}
FinalData{id='fd1', valueOne=9, i4=15,INT_5=18}
FinalData{id='fd2', valueOne=9, i4=13,INT_5=18}
```

- `final`参数
  - java 允许在参数列表中以声明的方式将参数指明为`final`。
  - 如果是参数也是一个引用，则无法在方法中更改参数引用所指向的对象；
  - 如果参数是一个基本类型，则在方法中只能读参数不能改参数。
  - 这一特性主要用来向匿名内部类访问局部变量时，局部变量必须使用`final`修饰。
  - Java 内部类实际会 copy 一份，而不是直接使用局部变量，使用`final`可以防止出现数据一致性问题。



# final 方法

使用`final`方法的原因：锁定方法，防止任何继承类修改其含义（确保在继承中使方法行为保持不变，且不会被覆盖）。所以只有在想要明确禁止覆盖时，才将方法设置为`final`。

类中所有的`private`方法都隐式地指定为是`final`的。由于无法取用`private`方法，自然也就无法覆盖它。但是，如果试图覆盖一个`private`方法的话，编译器不会给出错误信息，因为编译器将新的方法当作了一个全新的方法，而不是继承来的方法。

将方法声明为`final`，更重要的是这样做可以有效“关闭”动态绑定，即告诉编译期不需要对其进行动态绑定，这样，编译期就可以为`final`方法调用生成更有效的代码。不过这么做对于程序的整体性能并没有什么改观，所以最好根据设计来决定是否使用`final`，而不是处于试图提高性能的目的去使用。



# final类

使用`final`类，也就表明你不打算继承该类，且不允许别人这么做。即不希望该类有子类。

由于`final`类禁止继承，所以`final`类中所有方法都隐式指定为是`final`的。

