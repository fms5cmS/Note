# JDK1.8

见[尚学堂官网](http://www.sxt.cn/Java8-tutorial/java8_environment.html)

- `Optional`工具类：用于包含非空对象的容器对象

  ```java
  List<Student> students = new ArrayList<>();
  students.add(new Student(1L, "zzk", "male", 24));
  Optional.ofNullable(students) //判断 students 是否为null
      .orElse(Collections.emptyList()) //判断是否不为 null，是的话返回 Collections.emptyList()
      .forEach(System.out::println); //遍历过程中对每个元素执行 System.out::println 操作
  ```

  



## 1.1 Lambda表达式

- 函数式编程

  - 前提：接口只有一个方法

  - 使用函数式编程前：

    ```java
    //说明：Message是一个接口，其中有一个抽象方法fun()
    //匿名内部类实现接口：
    Message m =new Message(){
      public void fun(){
        System.out.println("HelloWorld！");
      }
    };
    m.fun();
    ```

  - 使用函数式编成后：

    ```java
    Message m = (形参) -> System.out.println("HelloWorld！");  //实现方法。单行时
    Message m = (形参) -> {                        //如果有多行，需要使用大括号
      System.out.println("HelloWorld！");
      System.out.println("HelloWorld！");
    } ;
    m.fun();
    ```

- `@FunctionInterface`注解标注的接口是一个函数式编程接口，只允许有一个抽象方法

  ```java
  @FunctionalInterface
  interface Message{
    void fun();
  }
  @FunctionalInterface
  interface TestReturn{
    int add(int a,int b);
  }
  public class LambdaTest {
    public static void main(String[] args) {
      Message m = () -> System.out.println("Hello");    //单行语句
      Message m2 = ()->{                                  //多行语句
        System.out.println("------------------------");
        System.out.println("多行语句");
      };
      m.fun();
      m2.fun();
      System.out.println("==================================");
      TestReturn t = (a,b)-> a+b; //直接返回值的话，可以不写return关键字；否则必须加{return ...}
      System.out.println(t.add(3, 4));
    }
  }
  ```

- Lambda表达式：可以替代特定匿名内部类；必须继承函数式接口

  - 语法：

    ```java
    基本：(参数列表) -> { statements; } 
    简化：(参数列表) -> expression
          单个参数 -> expression
    ```

    ```java
    //举例：
    (x,y) -> {
      x = x + y;
      y = x - y;
      x = x - y;
      return true;
    }
    ```



## 1.2 方法引用

  属于Lambda的补充，需要和Lambda表达式配合使用。

  代码中的接口均只有一个抽象方法,所以最好加`@FunctionInterface`

  - 引用静态方法：`类名称::static方法名称；`

    ```java
    IUtil<Integer,String> iu = String::valueOf;   //进行静态方法方法引用 I
    String str = iu.zhuanHuan(1000); //相当于str = String.valueOf(1000);
    ```

  - 引用某个对象的方法：`实例对象::普通方法名称；`

    ```java
    IUtil2<String> iu2 = "hello"::toUpperCase;  //引用了"hello"这个对象的toUpperCase()方法
    String str2 = iu2.zhuanHuan();         //做了将"hello"大写的操作
    ```

  - 引用某个特定类的方法：`类名称::普通方法名称；`

    ```java
    IUtil3<String,Integer> iu3 = String::compareTo;//引用特定类的方法
    int i = iu3.biJiao("a", "A");
    ```

  - 引用构造方法：`类名称::new； 调用默认构造器`



## 1.3 函数式接口

可以参考：https://www.oreilly.com/learning/java-8-functional-interfaces

`java.util.Function`包中定义的功能接口：。。。

接口都有注解：`@FunctionalInterface`

- 功能型函数式接口（输入一个数据，处理后再输出）：`public interface Function<T, R> {R apply(T t);}`

  ```java
  Function<Integer,String> fun = String :: valueOf; //使用带参方法
  System.out.println(fun.apply(100));    //输出一个100的字符串
  ```

  - 扩展的function接口，eg： `Interface IntFunction<R> {R apply(int value)}`

    ```java
    IntFunction<String> fun = String :: valueOf; //如果确定操作的数是int时，还有其他接口
    System.out.println(fun.apply(100));    //输出一个100的字符串
    ```

- 供给型函数式接口（付出不回报）：`Interface Supplier<T>{ public T get();}`

  ```java
  Supplier<String> sup = "hello".toUpperCase;    //使用不带参方法
  System.out.println(sup.get());    //输出HELLO
  ```

- 消费型函数式接口：`public interface Consumer<T>{ public void accept(T t)；}`

  ```java
  Consumer<String> cons = System.out :: println();   //使用带参方法
  cons.accept("HelloWorld!");    //输出HelloWOrld！
  ```

- 断言型函数式接口：`Interface Predicate<T>{ boolean test(T t)}`

  ```java
  Predicate<String> pre = "###hello"::startsWith; //使用带参方法
  System.out.println(pre.test("###"));    //输出true
  ```


## 1.4 接口定义加强

JDK1.8后，为了解决接口的子类、实现类非常多时的扩充问题

- 接口中:
  - 1.可以使用default来定义普通方法。需要通过对象调用；  即为默认方法
  - 2.可以使用static来定义静态方法。通过接口名就可以调用。

```java
interface IMessage{
  void fun();        //实现类需要实现的抽象方法
  default void fun1() {    //实现类会继承该方法
    //...有方法体
  }
  static int fun2() {    //定义了静态方法，可以有接口名直接调用
    //...有方法体
    return 0;
  }
}
```



- 当一个类实现了多个接口，而多个接口中不止一个接口有同名的默认方法时，解决办法：
  - 1.创建一个自己的方法，并覆盖默认实现；
  - 2.使用指定接口的默认方法。

```java
public class car implements vehicle, fourWheeler {
  default void print(){
    vehicle.super.print();    //使用指定接口的默认方法
  }
}
```



## 1.5 Stream

Stream 使用一种类似用 SQL 语句从数据库查询数据的直观方式来提供一种对 Java 集合运算和表达的高阶抽象，会把要处理的元素集合看作一种流，流在管道中传输，并且可以在管道的节点上进行处理，比如筛选，排序，聚合等。

- 特点：
  - 无存储。Stream不是一种数据结构，它只是某种数据源的一个视图，数据源可以是一个数组，Java容器或I/O channel等。
  - 为函数式编程而生。对Stream的任何修改都不会修改背后的数据源，比如对Stream执行过滤操作并不会删除被过滤的元素，而是会产生一个不包含被过滤元素的新Stream。
  - 惰式执行。Stream上的操作并不会立即执行，只有等到用户真正需要结果的时候才会执行。
  - 可消费性。Stream只能被“消费”一次，一旦遍历过就会失效，就像容器的迭代器那样，想要再次遍历必须重新生成。



### 创建流

有以下几种方式：

- 通过已有的集合来创建流：

  - JDk 1.8 对`Collection`接口做了增强，增加了`stream()`、`parallelStream()`方法来创建流或并行流

    ```java
    List<Student> students = new ArrayList<>();
    students.add(new Student(1L, "zzk", "male", 24));
    Stream<Student> stream = students.stream();
    Stream<Student> parallelStream = students.parallelStream();
    ```

- 通过`Stream`创建流：直接返回一个由指定元素组成的流

  ```java
  Stream<String> stream = Stream.of("a", "b", "c");
  ```



### 中间操作

Stream有很多中间操作(Intermediate operation)，多个中间操作可以连接起来形成一个流水线，每一个中间操作就像流水线上的一个工人，每人工人都可以对流进行加工，**加工后得到的结果还是一个流**。

- filter：通过设置的条件过滤出元素

  ```java
  List<String> strings = Arrays.asList("zzk", "", "bx", "H", "x");
  //通过filter过滤空字符串，filter接受一个函数作为参数
  strings.stream().filter(string -> !string.isEmpty()).forEach(System.out::println);
  ```

- map：一对一映射每个元素对应的结果

  ```java
  List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
  //输出元素对应的平方数
  numbers.stream().map( i -> i*i).forEach(System.out::println);
  ```

- limit 或 skip：

  - limit 返回 Stream 的前面 n 个元素
  - skip 是扔掉前 n 个元素

  ```java
  List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
  numbers.stream().limit(4).forEach(System.out::println);
  ```

- sorted：对流进行排序

  ```java
  List<Integer> numbers = Arrays.asList(3, 8, 2, 10, 7, 3, 5);
  numbers.stream().sorted().forEach(System.out::println);
  ```

- distinct：去重

  ```java
  List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
  numbers.stream().distinct().forEach(System.out::println);
  ```



示例：

```java
List<String> strings = Arrays.asList("Hollis", "HollisChuang", "hollis", "Hello", "HelloWorld", "Hollis");
Stream s = strings.stream()  //获取流
    //过滤字符串长度小于6的，结果为："Hollis"、"hollis"、"Hello"、"Hollis" 组成的 Stream
    .filter(string -> string.length()<= 6)
    //获取字符串的个数，结果为：6、6、5、6 组成的 Stream
    .map(String::length)
    //从小到大排序，结果为：5、6、6、6 组成的 Stream
    .sorted()
    //返回前三个元素，结果为：5、6、6 组成的 Stream
    .limit(3)
    //去重，结果为：5、6 组成的 Stream
    .distinct();
```



### 最终操作

最终操作(terminal operation)将一个`Stream`转换成需要的类型，如计算流中元素个数、将流转为集合等。

- forEach：迭代流中的每个数据

  ```java
  Random random = new Random();
  //ints() 返回一个有效的伪随机 int 值的流
  random.ints().limit(10).forEach(System.out::println);
  ```

- count：统计流中的元素个数

  ```java
  List<String> strings = Arrays.asList("zzk", "bx", "fS","ms5cm", "hen");
  System.out.println(strings.stream().count());  //5
  ```

- collect：一个归约操作，可以接受各种做法作为参数，将流中的元素累积成一个汇总结果

  - 使用`Collectors`来设置`collect()`的参数

  ```java
  List<String> s = Arrays.asList("zzk", "bx", "fS","ms5cm", "hen");
  //将以“z”开头的元素作为list收集起来
  //strings = s.stream().filter(string -> string.startsWith("z")).collect(Collectors.toList()); 
  //将不是""的元素以","作为分隔符收集起来
  strings = s.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
  System.out.println(strings); //zzk
  ```

