# AQS

`AbstractQueuedSynchronizer`（AQS）抽象队列同步器，在 java.util.concurrent.locks 包下。AQS定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的 `ReentrantLock`/`Semaphore`/`CountDownLatch`...。 

**AQS核心思想是，如果被请求的共享资源空闲，则将当前请求资源的线程设置为有效的工作线程，并且将共享资源设置为锁定状态。如果被请求的共享资源被占用，那么就需要一套线程阻塞等待以及被唤醒时锁分配的机制，这个机制AQS是用CLH队列锁实现的，即将暂时获取不到锁的线程加入到队列中。**

> CLH(Craig,Landin,and Hagersten)队列是一个虚拟的双向队列（虚拟的双向队列即不存在队列实例，仅存在结点之间的关联关系）。AQS是将每条请求共享资源的线程封装成一个CLH锁队列的一个结点（Node）来实现锁的分配。



**AQS定义了两种资源共享方式**：

- Exclusive（独占）：只有一个线程能执行，如`ReentrantLock`。又可分为公平锁和非公平锁：
- **Share**（共享）：多个线程可同时执行，如`Semaphore`/`CountDownLatch`。

`ReentrantReadWriteLock`可以看成是组合式的，因为`ReentrantReadWriteLock`允许多个线程同时对某一资源进行读。

不同的自定义同步器争用共享资源的方式也不同。自定义同步器在实现时只需要实现共享资源 state 的获取与释放方式即可，至于具体线程等待队列的维护（如获取资源失败入队/唤醒出队等），AQS已经在上层已经帮我们实现好了。



**AQS底层使用模板方法模式**

同步器的设计是基于模板方法模式的，如果需要自定义同步器一般的方式是这样（模板方法模式很经典的一个应用）：

1. 使用者继承`AbstractQueuedSynchronizer`并重写指定的方法。（重写方法是对于共享资源state的获取和释放）
   - `isHeldExclusively()`：该线程是否正在独占资源。只有用到condition才需要去实现它。
   - `tryAcquire(int)`：独占方式。尝试获取资源，成功则返回true，失败则返回false。
   - `tryRelease(int)`：独占方式。尝试释放资源，成功则返回true，失败则返回false。
   - `tryAcquireShared(int)`：共享方式。尝试获取资源。负数表示失败；0表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。
   - `tryReleaseShared(int)`：共享方式。尝试释放资源，成功则返回true，失败则返回false。
2. 将AQS组合在自定义同步组件的实现中，并调用其模板方法，而这些模板方法会调用使用者重写的方法。

以`ReentrantLock`为例，state初始化为0，表示未锁定状态。A线程l`ock()`时，会调用`tryAcquire()`独占该锁并将state+1。此后，其他线程再`tryAcquire()`时就会失败，直到A线程`unlock()`到state=0（即释放锁）为止，其它线程才有机会获取该锁。当然，释放锁之前，A线程自己是可以重复获取此锁的（state会累加），这就是可重入的概念。但要注意，获取多少次就要释放多么次，这样才能保证state是能回到零态的。

以`CountDownLatch`以例，任务分为N个子线程去执行，state也初始化为N（注意N要与线程个数一致）。这N个子线程是并行执行的，每个子线程执行完后`countDown()`一次，state会CAS(Compare and Swap)减1。等到所有子线程都执行完后(即state=0)，会`unpark()`主调用线程，然后主调用线程就会从`await()`函数返回，继续后面动作。

一般来说，自定义同步器要么是独占方法，要么是共享方式，他们也只需实现`tryAcquire-tryRelease`、`tryAcquireShared-tryReleaseShared`中的一种即可。但AQS也支持自定义同步器同时实现独占和共享两种方式，如`ReentrantReadWriteLock`。



参考：

- https://www.cnblogs.com/waterystone/p/4920797.html
- <https://www.cnblogs.com/chengxiao/archive/2017/07/24/7141160.html>

