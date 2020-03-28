# Q

Q：`final`、`finally`、`finalize`的不同。

A：

`final`可用来修饰类、方法、变量，分别有不同意义，`final`修饰的类代表不可继承扩展，`final`修饰的变量不可修改，`final`修饰的方法不可重写(override)；

`finally`是 Java 保证重点代码一定要被执行的一种机制，可以使用 try-finally、try-catch-finally 来进行关闭 JDBC 连接、保证 unlock 锁等动作一定会被执行，需要关闭的连接等资源，更推荐使用 Java 7中添加的 try-with-resources语句；

`finalize`是`java.lang.Object`的一个方法，设计目的是保证对象在被垃圾收集前完成特定资源的回收，不推荐使用，且在 JDK 9 开始被标记为`deprecated`。（Java 平台目前在逐步使用`java.lang.Cleaner`来替换原有的`finalize`实现，每个 Cleaner 的操作都是独立的，有自己的运行线程，可以避免意外死锁等问题。不要完全依赖 Cleaner 进行资源回收！）



# 扩展

`final`变量产生了一定程度的不可变(immutable)效果，所以可用于保护只读数据，尤其是在并发编程中，因为明确地不能再赋值 `final`变量，有利于减少额外的同步开销，也可省去一些防御性拷贝的必要。



## 实现 immutable 的类

`final`并不是 immutable！如果要实现 immutable 的类，需要：

1. 类声明为`final`；
   - 可以有效避免 API 使用者更改基础功能，某种程度上，这是保证平台安全的必要手段。
2. 所有成员变量定义为`private`和`final`，且不要实现 setter 方法；
3. 通常构造对象时，成员变量使用深度拷贝来初始化，而不是直接赋值，这是一种防御措施，因为你无法确定输入对象不被其他人修改；
4. 如果确实要实现 getter 方法，或其他可能会返回内部状态的方法，使用 copy-on-write 原则，创建私有的 copy。
