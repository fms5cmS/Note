# CountDownLatch

`CountDownLatch`是一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。

维护了一个计数器 cnt，每次调用 `countDown() `方法会让计数器的值减 1，减到 0 的时候，那些因为调用 `await() `方法而在等待的线程就会被唤醒。

三种用法：

- 某一线程在开始运行前等待n个线程执行完毕。将`CountDownLatch`的计数器初始化为n ：`new CountDownLatch(n) `，每当一个任务线程执行完毕，就将计数器减1 `countdownlatch.countDown()`，当计数器的值变为0时，在`CountDownLatch上 await()`的线程就会被唤醒。
  - 一个典型应用场景就是启动一个服务时，主线程需要等待多个组件加载完毕，之后再继续执行。
- 实现多个线程开始执行任务的最大并行性。注意是并行性，不是并发，强调的是多个线程在某一时刻同时开始执行。类似于赛跑，将多个线程放到起点，等待发令枪响，然后同时开跑。做法是初始化一个共享的 `CountDownLatch` 对象，将其计数器初始化为 1 ：`new CountDownLatch(1) `，多个线程在开始执行任务前首先 `coundownlatch.await()`，当主线程调用`countDown()`时，计数器变为0，多个线程同时被唤醒。
- 死锁检测：一个非常方便的使用场景是，可以使用n个线程访问共享资源，在每次测试阶段的线程数目是不同的，并尝试产生死锁。

```java
public class CountDownLatchExample {
  // 请求的数量
  private static final int threadCount = 550;

  public static void main(String[] args) throws InterruptedException {
    // 创建一个具有固定线程数量的线程池对象
    ExecutorService threadPool = Executors.newFixedThreadPool(300);
    final CountDownLatch countDownLatch = new CountDownLatch(threadCount);
    for (int i = 0; i < threadCount; i++) {
      final int threadnum = i;
      threadPool.execute(() -> {
        try {
          test(threadnum);
        } catch (InterruptedException e) {
          e.printStackTrace();
        } finally {
          countDownLatch.countDown();// 表示一个请求已经被完成
        }

      });
    }
    countDownLatch.await();
    threadPool.shutdown();
    System.out.println("finish");
  }

  public static void test(int threadnum) throws InterruptedException {
    Thread.sleep(1000);// 模拟请求的耗时操作
    System.out.println("threadnum:" + threadnum);
    Thread.sleep(1000);// 模拟请求的耗时操作
  }
}
```

注意：`CountDownLatch`是一次性的，计数器的值只能在构造方法中初始化一次，之后没有任何机制再次对其设置值，当`CountDownLatch`使用完毕后，它不能再次被使用。

# CyclicBarrier 

`CyclicBarrier`用来控制多个线程互相等待，只有当多个线程都到达时，这些线程才会继续执行。和 `CountdownLatch` 相似，都是通过维护计数器来实现的，两者区别：

- 计数器是递增的，每次执行 `await() `方法之后，计数器会加 1，直到计数器的值和设置的值相等，等待的所有线程才会继续执行。
- `CyclicBarrier` 的计数器可以循环使用，所以它才叫做循环屏障。 

`CyclicBarrier`可以用于多线程计算数据，最后合并计算结果的应用场景。

```java
public class CyclicBarrierExample {
  // 请求的数量
  private static final int threadCount = 550;
  //参数表示屏障拦截的线程数量，每个线程调用await方法告诉 CyclicBarrier 我已经到达了屏障，然后当前线程被阻塞
  private static final CyclicBarrier cyclicBarrier = new CyclicBarrier(5);

  public static void main(String[] args) throws InterruptedException {
    // 创建线程池
    ExecutorService threadPool = Executors.newFixedThreadPool(10);

    for (int i = 0; i < threadCount; i++) {
      final int threadNum = i;
      Thread.sleep(1000);
      threadPool.execute(() -> {
        try {
          test(threadNum);
        } catch (InterruptedException e) {
          e.printStackTrace();
        } catch (BrokenBarrierException e) {
          e.printStackTrace();
        }
      });
    }
    threadPool.shutdown();
  }

  public static void test(int threadnum) throws InterruptedException, BrokenBarrierException {
    System.out.println("threadnum:" + threadnum + "is ready");
    try {
      /**等待60秒，保证子线程完全执行结束*/  
      cyclicBarrier.await(60, TimeUnit.SECONDS);
    } catch (Exception e) {
      System.out.println("-----CyclicBarrierException------");
    }
    System.out.println("threadnum:" + threadnum + "is finish");
  }
}
```

`CyclicBarrier`还提供一个更高级的构造函数`CyclicBarrier(int parties, Runnable barrierAction)`，用于在线程到达屏障时，优先执行`barrierAction`，方便处理更复杂的业务场景。

# Semaphore

Semaphore（信号量） 是一个线程同步结构，用于在线程间传递信号，以避免出现信号丢失，或者像锁一样用于保护一个关键区域。JDK 1.5开始在并发包中提供了其实现：`Semaphore` ，可以控制对互斥资源的访问线程数，指定多个线程同时访问某个资源。

```java
//模拟了对某个服务的并发请求，每次只能有 3 个客户端同时访问，请求总数为 10。
public class SemaphoreExample {
  public static void main(String[] args) {
    final int clientCount = 3;
    final int totalRequestCount = 10;
    Semaphore semaphore = new Semaphore(clientCount);
    ExecutorService executorService = Executors.newCachedThreadPool();
    for (int i = 0; i < totalRequestCount; i++) {
      executorService.execute(()->{
        try {
          semaphore.acquire();
          System.out.print(semaphore.availablePermits() + " ");
        } catch (InterruptedException e) {
          e.printStackTrace();
        } finally {
          semaphore.release();
        }
      });
    }
    executorService.shutdown();
  }
}
结果：2 1 2 2 2 2 2 1 2 2
```



# FutureTask

`FutureTask` 实现了 `RunnableFuture 接口`，该接口继承自 `Runnable` 和 `Future` 接口，这使得 `FutureTask` 既可以当做一个任务执行，也可以有返回值。 

`FutureTask `可用于异步获取执行结果或取消执行任务的场景。当一个计算任务需要执行很长时间，那么就可以用` FutureTask` 来封装这个任务，主线程在完成自己的任务之后再去获取结果。 

```java
public class FutureTaskExample {
  public static void main(String[] args) throws ExecutionException, InterruptedException {
    FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
      @Override
      public Integer call() throws Exception {
        int result = 0;
        for (int i = 0; i < 100; i++) {
          Thread.sleep(10);
          result += i;
        }
        return result;
      }
    });

    Thread computeThread = new Thread(futureTask);
    computeThread.start();

    Thread otherThread = new Thread(() -> {
      System.out.println("other task is running...");
      try {
        Thread.sleep(1000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
    });
    otherThread.start();
    System.out.println(futureTask.get());
  }
}
结果：
other task is running...
  4950
```



# BlockingQueue

`java.util.concurrent.BlockingQueue` 接口有以下阻塞队列的实现：

- **FIFO 队列** ：`LinkedBlockingQueue`、`ArrayBlockingQueue`（固定长度）
- **优先级队列** ：`PriorityBlockingQueue`

提供了阻塞的` take()` 和 `put() `方法：如果队列为空 `take() `将阻塞，直到队列中有内容；如果队列为满` put() `将阻塞，直到队列有空闲位置。

