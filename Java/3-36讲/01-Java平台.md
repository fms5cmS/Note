

# Q1

Q：谈谈对Java平台的理解。

A：Java是一种面向对象的语言，显著特性：

1. “Compile once , run anywhere”，能够容易地获得跨平台能力，其实并不是 Java 语言可以跨平台，而是在不同平台都有可以让 Java 语言运行的环境，所以才有了 Java 一次编译到处运行的效果；
2. 垃圾收集(GC,Garbage Collection)，Java 通过垃圾收集器(Garbage Collector) 回收分配内存，大部分情况下，不需要关心内存的分配和回收。

还可以从很多方面简要谈，如：面向对象(封装、继承、多态)、内存模型、Java 语言特性(反射、泛型、Lambda等)、基础类库(集合、utils、IO/NIO、网络、并发、安全等基础类库)，也可谈谈 JVM 的一些基础概念和机制。



# Q2

Q：“Java 是解释执行”这句话正确吗?

A：这个说法不准确。Java 源代码先通过 Javac 编译成为字节码(bytecode)，然后，在运行时，通过 JVM 内嵌的解释器将字节码转换成为最终的机器码。但是常见的 JVM，如大多数情况使用的 Oracle JDK 提供的 Hotspot JVM，都提供 JIT(Just-In-Time) 编译器，即动态编译器，JIT 能在运行时将热点代码(高频调用的方法和代码块)编译成机器码，这样类似于缓存技术，运行时再遇到这类代码直接可以执行，而不是先解释后执行，这种情况下部分热点代码就属于编译执行，而不是解释执行。



# 扩展

Java 的编译和 C/C++ 是不同的，Javac 的编译，编译 Java 源码生成".class"文件，里面实际是字节码，而不是可直接执行的机器码，Java 通过字节码和 JVM 这种跨平台的抽象，屏蔽了操作系统和硬件的细节，这也是实现“Compile once , run anywhere”的基础。

运行时，JVM 会通过 Class-Loader 加载字节码，解释或编译执行。如 JDK8 实际是解释和编译混合的一种模式，即所谓的混合模式 (-Xmixed)。通常运行在 server 模式的JVM，会进行上万次调用以收集足够信息进行高效的编译，client 模式的这个门限是 1500 次。Oracle Hotspot JVM 内置了两个不同 JIT compiler，C1 对应 client 模式，适用于对启动速度敏感的应用(如普通 Java 桌面应用)；C2 对应 server 模式，它的优化是为长时间运行的服务器端应用设计的。默认采用所谓的分层编译(TieredCompilation)。

JVM 启动时，可指定不同参数对运行模式进行选择。如执行`-Xint`就是告诉 JVM 只进行解释执行，部队代码进行编译，这种模式抛弃了 JIT 可能带来的性能优势，因为解释器(interpreter)是逐条读入逐、条解释运行的。还有一个`-Xcomp`参数是告诉 JVM 关闭解释器，不进行解释执行，也叫做最大优化级别，会导致 JVM 启动变慢很多。

一种新的编译方式：AOT（Ahead-of-Time Compilation），在运行前，通过工具直接将字节码编译成机器代码，这样就避免了 JIT 预热等各方面的开销，如 Oracle JDK9 引入了实验性的 AOT 特性，并增加了新的 jaotc 工具。













