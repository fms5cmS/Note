# Q

Q：`synchronized`和`ReentrantLock`有什么区别？有人说`synchronized`最慢，这话靠谱吗？

A：`synchronized`是Java内建的同步机制，提供了互斥的语义和可见性，当一个线程已经获取当前锁时，其他视图获取的线程只能等待或阻塞在那里。在代码中`synchronized`可用于修饰方法，也可是使用在特定代码块上。

`ReentrantLock`是可重入锁，语义和`synchronized`基本相同。可重入锁通过代码直接调用`lock()`获取，书写灵活，且提供了很多使用的方法，能实现很多`synchronized`无法做到的细节控制，如控制 fairness 即公平性，或利用定义条件等。注意，必须明确调用`unlock()`释放，否则会一直持有锁。

早期版本`synchronized`在很多场景下性能相差很大，在后续版本中进行了较多改进，在低竞争场景中表现可能优于`ReetrantLock`。



# 扩展

掌握：

- 理解什么是线程安全。
- `synchronized`、`ReentrantLock`等机制的基本使用与案例。
- 掌握`synchronized`、`ReentrantLock`底层实现；理解锁膨胀、降级；理解偏斜锁、自旋锁、轻量级锁、重量级锁等概念。
- 掌握并发包中`java.util.concurrent.lock`各种不同实现和案例分析。



## 线程安全

线程安全是一个多线程环境下正确性的概念，也就是保证多线程环境下共享的、可修改的状态的正确性，这里的状态反映在程序中其实可以看作是数据。换个角度来看，如果状态不是共享的，或者不是可修改的，也就不存在线程安全问题，进而可以推理出保证线程安全的两个办法：

- 封装：通过封装，我们可以将对象内部状态隐藏、保护起来。
- 不可变。

线程安全需要保证几个基本特性：

- 原子性，简单说就是相关操作不会中途被其他线程干扰，一般通过同步机制实现。
- 可见性，是一个线程修改了某个共享变量，其状态能够立即被其他线程知晓，通常被解释为将线程本地状态反映到主内存上，`volatile`就是负责保证可见性的。
- 有序性，是保证线程内串行语义，避免指令重排等。



## ReentrantLock

可重入：表示当一个线程试图获取一个它已经获取的锁时，这个获取动作就自动成功，这是对锁获取粒度的一个概念，也就是锁的持有是以线程为单位而不是基于调用次数。Java 锁实现强调再入性是为了和 pthread 的行为进行区分。

- 可重入锁可以设置公平性：

  - 这里公平性是指在竞争场景中，当公平性为真时，会倾向于将锁赋予等待时间最久的线程。公平性是减少线程“饥饿”（个别线程长期等待锁，但始终无法获取）情况发生的一个办法。

  ```java
  ReentrantLock fairLock = new ReentrantLock(true); //通过参数true来选择公平
  ```

  - 通用场景中，公平性未必有想象中的那么重要，Java 默认的调度策略很少会导致 “饥饿”发生。与此同时，若要保证公平性则会引入额外开销，自然会导致一定的吞吐量下降。建议只有当程序确实有公平性需要的时候，才有必要指定它。

- 带超时的获取锁尝试

- 可以判断是否有线程，或者某个特定线程，在排队等待获取锁

- 可以响应中断请求

- 条件变量(`java.util.concurrent.Condition`)：如果说`ReentrantLock`是`synchronized`的替代选择，`Condition`则是将`wait`、`notify`、`notifyAll`等操作转化为相应的对象，将复杂而晦涩的同步操作转变为直观可控的对象行为。

  - 典型应用场景：标准类库中的`ArrayBlockingQueue`等，下面是部分`ArrayBlockingQueue`源码：

    ```java
        /** Condition for waiting takes */
        private final Condition notEmpty;
    
        /** Condition for waiting puts */
        private final Condition notFull;
    
        public ArrayBlockingQueue(int capacity, boolean fair) {
            if (capacity <= 0)
                throw new IllegalArgumentException();
            this.items = new Object[capacity];
            lock = new ReentrantLock(fair);
            //通过同一个可重入锁获取条件变量
            notEmpty = lock.newCondition();
            notFull =  lock.newCondition();
        }
    	//从队列中获取队首元素
        public E take() throws InterruptedException {
            final ReentrantLock lock = this.lock;
            lock.lockInterruptibly();
            try {
                while (count == 0) //队列为空时
                    notEmpty.await(); //试图take的线程的正确行为是等待入队发生，而不是直接返回
                return dequeue();
            } finally {
                lock.unlock();
            }
        }
    	//入队操作
        private void enqueue(E x) {
            // assert lock.getHoldCount() == 1;
            // assert items[putIndex] == null;
            final Object[] items = this.items;
            items[putIndex] = x;
            if (++putIndex == items.length)
                putIndex = 0;
            count++;
            notEmpty.signal();  //通知等待的线程“非空条件已经满足”，于是上面等待的take线程就可以从队列中获取元素了
        }
    ```

  - 通过`signal`/`await`的组合，完成了条件判断和通知等待线程，非常顺畅就完成了状态流转。注意，`signal`和`await`成对调用非常重要，不然假设只有`await`动作，线程会一直等待直到被打断（interrupt）



