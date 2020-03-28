# 高精度数字

Java 中普通整型是有长度限制的，浮点型运算的精度不足无法用于比较，所以为了计算任意大小的整数和浮点数比较，Java 提供了两个用于高精度计算的类：`BigInteger`和`BigDecimal`，它们包含的方法和基本数据类型所能做的操作类似。由于是调用方法来取代运算符方式完成的，所以运算速度相比基本数据类型会慢一些，是一种以速度换取精度的的方式。

`BigInteger`可以准确表示任意大小的整数值，而不会丢失信息；`BigDecimal`支持任意精度的浮点数，可以用来进行浮点数的比较。具体使用见 API 文档。



# 枚举类

使用`enum`关键字创建的一种类。在后台实现时，实际上是转化为一个继承了`java.lang.Enum`类的实体类，原先的枚举类型变成对应的实体类型。 

```java
enum Prices{
    FIRST,SECOND,THIRD,FOUR;//枚举类型的实例是常量，使用大写字母表示，如果一个名字里有多个单词，用_分隔
}
```

创建 `enum`时，编译器会自动添加一些有用的特性。如：创建`toString()`方法；创建`ordinal()`方法，用来表示某个特定`enum`常量的声明顺序；创建`static values()`方法，可以按照常量的声明顺序，产生由常量构成的数组。见下面的测试方法：

```java
    @Test
    public void testEnum1(){
        Prices p = Prices.FIRST; //获取枚举类的一个实例
        System.out.println(p);  
        for(Prices pr:Prices.values()){
            System.out.println(pr + " " + pr.ordinal());
        }
    }
```

```java
//输出结果
FIRST
FIRST 0
SECOND 1
THIRD 2
FOUR 3
```

枚举类也可以实现接口：

```java
//枚举类；实现接口的枚举值
public enum Season implements TimeInfo{ //该接口仅有一个 getTimeInfo()方法

  //必须在枚举类的第一行写出有哪些枚举值.(自动调用构造器)
  //若需要每个枚举值在调用实现的接口方法呈现不同的行为方式，可以让每个枚举值分别实现该方法
  SPRING("春天", "春风又绿江南岸"){
    @Override
    public void getTimeInfo() {
      System.out.println("2-5月");
    }
  },
  SUMMER("夏天", "映日荷花别样红"){
    @Override
    public void getTimeInfo() {
      System.out.println("5-8月");
    }
  },
  AUTUMN("秋天", "秋水共长天一色"){
    @Override
    public void getTimeInfo() {
      System.out.println("8-11月");
    }
  },
  WINTER("冬天", "窗含西岭千秋雪"){
    @Override
    public void getTimeInfo() {
      System.out.println("11-2月");
    }
  };

  //2.因为对象固定，所以属性是常量
  private final String name;
  private final String desc;
  //1.因为枚举类的对象有限，所以不能在类的外部创建类的对象。私有化构造器
  private Season(String name, String desc) {
    this.name = name;
    this.desc = desc;
  }
  public String getName() {
    return name;
  }
  public String getDesc() {
    return desc;
  }
}
```



# 比较器的使用

面试中，如果题目的解答中排序仅仅是某一步，为了快速完成排序，可以使用比较器。

让一个类实现`java.util.Comparator<T>`接口，然后重写`compare(T o1,T o2)`方法（自定义比较规则）。这样就使得排序类与实体类解耦，可以应对不同的排序规则。`compare(T o1,T o2)`方法：如果返回值为负数，则o1排在o2前面；返回值为0，o1等于o2；返回值为正数，o1排在o2后面。

对引用类型排序，

`Arrays.sort(Object[] a)`默认根据数组中元素的内存地址来排序；

`Arrays.sort(T[] a, Comparator<? super T> c)`会根据第二个参数(比较器)指定的比较规则来对第一个参数(数组)排序。



# 注意

1. 父类没有无参的构造函数，所以子类需要在自己的构造函数中显式调用父类的构造函数。
2. 子类有新增方法时，不能使用多态。
3. switch 不支持 long，是因为 swicth 的设计初衷是为那些只需要对少数的几个值进行等值判断，如果值过于复杂，那么还是用 if 比较合适。 
