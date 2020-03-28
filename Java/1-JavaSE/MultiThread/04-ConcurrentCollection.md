一、Map/Set的选择：

` ConcurrentHashMap`采用“锁分段”机制（JDK1.8以前，1.8以后也采用了CAS算法    ）

其中`Concurrent Level`默认为16，即16段`Segment`，每段中Hash表长度为16.每一段都有独立的锁。当有多个线程访问时，不同线程可以访问不同段，实现并行，效率较高。 

==当期望许多线程访问一个给定的`Collection`时，``ConcurrentHashMap``通常优于同步的`HashMap`==；        	

==`ConcurrentSkipListMap`（高并发且排序时）通常优于同步的`TreeMap`，`ConcurrentSkipListMap`在插入时效率较低，查找时会快很多==。

==当期望的读数和遍历远远大于列表的更新数时，`CopyOnWriteArrayList`优于同步的ArayList==。 



二、队列 :

写时复制容器`copy on write`，多线程环境下，写时效率低，读时效率高，适合写少读多的环境。

` CopyOnWriteArrayList`每次写入时都会复制一段新链表再添加，所以不适合添加操作多的场景，更适合用于并发迭代操作。读的时候不会发生脏读的情况，所以读的时候不需要加锁 。

- Ⅰ并发量低时：
  - `ArrayList`
  - `LinkedList`
  - `Collections.SynchronizedXxx `
- Ⅱ并发量高时`Queue`：并发中应用最多的容器 
  - `ConcurrentLinkedQueue` 
  - `BlockingQueue `
    - 1）`LinkedBlockingQueue`   无界队列 
    - 2）`ArrayBlockingQueue`     容量有限（自己指定）
    - 3）`SynchronousQueue `   容量为0，来的任何东西，消费者必须立刻消费掉。不能用`add()`方法，要使用`put()`方法添加，此时，会阻塞等待消费者消费
    - 4）`TransferQueue`更高的并发情况下使用
    - ` LinkedTransferQueue`有一个`transfer()`方法。首先启动消费者线程，然后启动生产者线程，生产出来的东西先不往队列中放，而是先看有没有消费者，如果有，直接给消费者；没有的话，线程会在`transfer()`方法处阻塞。如果不用`transfer()`而用`put()` / `add()` / `offer()` 方法，不会发生上面的情况。
  - `DelayQueue` 无界队列，里面的元素只有在等待了指定时间后才能被消费者线程拿出去。可用于定时执行任务 

