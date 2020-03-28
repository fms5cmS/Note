使用线程池的好处：

1. 降低资源消耗。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
2. 提高响应速度。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
3. 提高线程的可管理性。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。 

`Executor` 框架是` Java5 `之后引入，通过 `Executor` 来启动线程比使用 `Thread` 的 `start()`方法更好，更易管理，且效率更好（使用线程池实现，节约开销），有助于避免`this`逃逸问题。

 `this`逃逸是指在构造函数返回之前其他线程就持有该对象的引用. 调用尚未构造完全的对象的方法可能引发令人错误。

- **任务**：`Runnalbe`或`Callabel`接口的实现类
- **接口**：
  - ` Executor`接口：只有一个`execute()`方法，执行没有返回值的任务；
  - `ExecutorService`接口：继承自`Executor`接口，除了`execute()`方法，还有一个`submit()`方法来执行任务。而`submit()`方法既可以执行没有返回值的任务，也可以执行有返回值的任务。
  - `Future`接口
- **类**：
  - `ThreadPoolExecutor`实现了`ExecutorService`接口
  - `ScheduledThreadPoolExecutor` 实现了`ExecutorService`接口 
  - `FutureTask`类实现了  `Future`接口和`Runnable`接口。详见 JUC 部分
- **常用方法**：
  - `execute()`方法用于提交不需要返回值的任务，所以无法判断任务是否被线程池执行成功与否。 
  - `submit(Callable<T> task) ` 提交一个实现`Callable接口`的任务，并且返回封装了异步计算结果的 `Future`。
  - `submit(Runnable task, T result) `提交一个实现`Runnable接口`的任务，并且指定了在调用 `Future`的`get()`方法时返回的result对象。
  - `submit(Runnable task) `提交一个实现`Runnable接口`的任务，并且返回封装了异步计算结果的 `Future `。
  - `shutdown() `正常关闭线程池，会等待所有任务执行完再关闭，且表明关闭已在`Executor`上调用，因此不会再向`DelayedPool`添加任何其他任务 。
  - `shutdownNow()`直接关闭线程池 。
  - `isTerminated() `判断所有任务是否都执行完了。 
  - `isShutdown() `判断线程池是否关闭了 。

**使用**

1. 创建实现了`Runnable`和`Callable`接口的任务对象；
2. 将创建后的对象交给`ExecutorService`执行（调用`execute()`或`submit()`方法）；
3. 如果执行的是`ExecutorService.submit()`方法，会返回一个实现了`Future`接口的对象；
4. 主线程可以执行`FutureTask.get()`方法来等待任务执行完成。主线程也可以执行`FutureTask.cancel（boolean mayInterruptIfRunning）`来取消此任务的执行。





# 创建线程池

《阿里巴巴Java开发手册》中强制线程池不允许使用`Executors`去创建，而是通过`ThreadPoolExecutor`的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。

> Executors 返回线程池对象的弊端：
>
> - **FixedThreadPool 和 SingleThreadExecutor** ： 允许请求的队列长度为`Integer.MAX_VALUE`,可能堆积大量的请求，从而导致OOM。
> - **CachedThreadPool 和 ScheduledThreadPool** ： 允许创建的线程数量为`Integer.MAX_VALUE`，可能会创建大量线程，从而导致OOM。

- 通过`ThreadPoolExecutor`的构造器实现；
- 通过`Executor`框架的工具类`Executors`来实现（下面`ThreadPoolExecutor`就是这么实现的）。



# ThreadPoolExecutor

通过工具类`Executors`来创建三种`ThreadPoolExecutor`:

- `FixedThreadPool`
- `SingleThreadPool`
- `CachedThreadPool`

几个线程池的核心参数：

- `corePoolSize` 为线程池的基本大小。
- `maximumPoolSize` 为线程池最大线程大小。
- `keepAliveTime` 和 `unit` 则是线程空闲后的存活时间。
- `workQueue` 用于存放任务的阻塞队列。
- `handler` 当队列和最大线程池都满了之后的饱和策略。



## FixedThreadPool

`FixedThreadPool`是可重用固定线程数的线程池。线程池的大小一旦达到最大值就会保持不变，如果某个线程因为执行异常而结束，那么线程池会补充一个新线程。 

每次扔进去一个任务，就会有一个空闲的线程去执行，当任务数大于线程数，又没有线程空闲，则任务会在任务队列`LinkedBlockingQueue`中等待，直到有线程执行完任务后空闲下来才会去执行这个任务 。

如果不`shutdown`，线程池就会一直空等着。 

**`FixedThreadPool`使用无界队列 `LinkedBlockingQueue`（队列的容量为`Intger.MAX_VALUE`）作为线程池的工作队列会对线程池带来如下影响：**

1. 当线程池中的线程数达到`corePoolSize`后，新任务将在无界队列中等待，因此线程池中的线程数不会超过`corePoolSize`；
2. 由于1，使用无界队列时`maximumPoolSize`将是一个无效参数；
3. 由于1和2，使用无界队列时`keepAliveTime`将是一个无效参数；
4. 运行中的`FixedThreadPool`（未执行`shutdown()`或`shutdownNow()`方法）不会拒绝任务

```java
public class T05_ThreadPool {
  public static void main(String[] args) throws InterruptedException {
    ExecutorService service = Executors.newFixedThreadPool(5);  //指定线程数为5
    for(int i = 0;i < 6;i++) {    //6个任务，多于池中的线程数
      service.execute(() -> {
        try {
          TimeUnit.MILLISECONDS.sleep(500);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName());
      });
    }
    System.out.println(service);

    service.shutdown();  //正常关闭线程池，会等待所有任务执行完再关闭.
    //service.shutdownNow();  //直接关闭线程池
    System.out.println(service.isTerminated());  //判断所有任务是否都执行完了
    System.out.println(service.isShutdown()); //true 是正在关闭中
    System.out.println(service);

    TimeUnit.SECONDS.sleep(5);
    System.out.println(service.isTerminated());
    System.out.println(service.isShutdown());  //true 是已经关闭了
    System.out.println(service);
  }
}
```

```java
java.util.concurrent.ThreadPoolExecutor@119d7047[Running, pool size = 5, active threads = 5, queued tasks = 1, completed tasks = 0]
false
true
java.util.concurrent.ThreadPoolExecutor@119d7047[Shutting down, pool size = 5, active threads = 5, queued tasks = 1, completed tasks = 0]
pool-1-thread-2
pool-1-thread-5
pool-1-thread-4
pool-1-thread-3
pool-1-thread-1
pool-1-thread-2
true
true
java.util.concurrent.ThreadPoolExecutor@119d7047[Terminated, pool size = 0, active threads = 0, queued tasks = 0, completed tasks = 6]
```



## CachedThreadPool 

会根据需要创建新线程；

创建一个可缓存的线程池。如果线程池的大小超过了处理任务所需要的线程，那么就会回收部分空闲的线程，当任务数增加时，此线程池又添加新线程来处理任务。 

一开始一个线程也没有，来一个任务起一个线程，如果有空闲线程，就不用起一个新的线程了。 

默认的每个线程空闲超过60s就会自动销毁 。

```java
public class T08_CachedPool {
  public static void main(String[] args) throws InterruptedException {
    ExecutorService service = Executors.newCachedThreadPool();
    System.out.println(service);

    for(int i=0;i<2;i++) {
      service.execute(() -> {
        try {
          TimeUnit.MILLISECONDS .sleep(500);
        } catch (InterruptedException e) {
          e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName());
      });
    }

    System.out.println(service);      
    TimeUnit.SECONDS.sleep(80);         
    System.out.println(service);
  }
}
```

```java
java.util.concurrent.ThreadPoolExecutor@330bedb4[Running, pool size = 0, active threads = 0, queued tasks = 0, completed tasks = 0]
java.util.concurrent.ThreadPoolExecutor@330bedb4[Running, pool size = 2, active threads = 2, queued tasks = 0, completed tasks = 0]
pool-1-thread-2
pool-1-thread-1
```



## SingleThreadPool 

线程池中只有一个线程 。任务队列也是`LinkedBlockingQueue `。

可以保证任务前后一定是顺序执行的，一个任务执行完以后才会执行下一个任务 。

```java
public class T09_SingleThreadPool {
  public static void main(String[] args) {
    ExecutorService service = Executors.newSingleThreadExecutor();
    for(int i=0;i<5;i++) {
      final int j = i;
      service.execute(() -> {
        System.out.println(j + " " + Thread.currentThread().getName());
      });
    }
  }
}
```



# ScheduledThreadPoolExecutor

`ScheduledThreadPoolExecutor`主要用来在给定的延迟后运行任务，或者定期执行任务。

`ScheduledThreadPoolExecutor`使用的任务队列`DelayQueue`封装了一个`PriorityQueue`，`PriorityQueue`会对队列中的任务进行排序，执行所需时间短的放在前面先被执行(`ScheduledFutureTask`的`time`变量小的先执行)，如果执行所需时间相同则先提交的任务将被先执行(`ScheduledFutureTask`的`squenceNumber`变量小的先执行)。



# SchedulePool 

支持定时以及周期性执行任务的需求。 

```java
public class T10_SchedulePool {
  public static void main(String[] args) {
    ScheduledExecutorService service = Executors.newScheduledThreadPool(4);
    service.scheduleAtFixedRate(() -> {    //以固定频率来执行某个任务
      try {
        TimeUnit.MILLISECONDS.sleep(new Random().nextInt(1000));
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println(Thread.currentThread().getName());
    }, 0, 500, TimeUnit.MILLISECONDS);//任务第一次执行是在0ms后，每隔500ms执行一次，单位为MILLISECONDS
  }
}
```



# WorkStealingPool

 JDK1.8后才有的。一个拥有多个任务队列的线程池，可以减少连接数，创建当前可用cpu数量的线程来并行执行。

会根据所需的并行层次来动态创建和关闭线程，通过使用多个队列减少竞争，底层使用`ForkJoinPool`来实现的。

`ForkJoinPool`的优势在于，可以充分利用多cpu，多核cpu的优势，把一个任务拆分成多个“小任务”，把多个“小任务”放到多个处理器核心上并行执行；当多个“小任务”执行完成之后，再将这些执行结果合并起来即可。

```java
/**
 * 任务窃取：
 * A线程的任务队列有5个，B线程有1个，C线程有2个，当B线程执行完任务队列的任务后，主动从A或C线程的任务队列中拿一个任务执行。
 * WorkStealingPool：
 * 1.根据CPU的核数来起不同个数的线程
 * 3.线程都是后台线程
 * 3.用于任务分配不均时
 */
public class T11_WorkStealingPool {
  public static void main(String[] args) throws IOException {
    ExecutorService service = Executors.newWorkStealingPool();  //默认根据CPU是几核来起不同个数的后台线程
    System.out.println(Runtime.getRuntime().availableProcessors()); //看系统是几核的

    //哪个线程空闲，将任务自动给谁。
    service.execute(new R(1000));
    service.execute(new R(2000));
    service.execute(new R(2000));
    service.execute(new R(2000));
    service.execute(new R(2000));
    service.execute(new R(2000));
    service.execute(new R(2000));
    service.execute(new R(2000));
    service.execute(new R(2000));
    service.execute(new R(2000));
    service.execute(new R(2000));

    //由于产生的是精灵线程（守护线程、后台线程），主线程不阻塞的话，看不到输出
    System.in.read();
  }

  static class R implements Runnable{
    int time;
    R(int t){
      this.time = t;
    }
    @Override
    public void run() {
      try {
        TimeUnit.MILLISECONDS.sleep(time);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      System.out.println(time + " " + Thread.currentThread().getName());
    }
  }
}
```



# ForkJoinPool 

```java
/**
 * ForkJoinPool：
 * 1.里面执行的任务一定是ForkJoinTask的，一般使用它的子类：RecursiveAction（无返回值）、RecursiveTask（有返回值）、CounterCompleter
 * 2.用于大规模计算
 */
public class T12_ForkJoinPool {
  static int[] nums = new int[100_0000];
  static final int MAX_NUM = 5_0000;//每个小任务计算的数值不超过50000
  static Random r = new Random();

  static {
    //方法一：直接求和
    for(int i = 0;i < nums.length;i++) {
      nums[i] = r.nextInt(100);
    }
    System.out.println(Arrays.stream(nums).sum());//stream api
  }
  /*
    static class AddTask extends RecursiveAction {
         int start,end;//计算的区间

         AddTask(int s,int e){
             start = s;
             end = e;
         }

         @Override
         protected void compute() {

             if(end-start <= MAX_NUM) {
                 long sum = 0L;
                 for(int i = start;i<end;i++) {
                     sum += nums[i];
                 }
                 //RecursiveAction没有返回值，只能打印出来看结果
                 System.out.println("from:" + start + "to:" + end + " = " + sum);
             }else {  //超出了最小的数值，继续将任务分隔得更小

                 int middle = start + (end-start)/2;

                 AddTask subTask1 = new AddTask(start, middle);
                 AddTask subTask2 = new AddTask(middle, end);
                 subTask1.fork(); //该任务再起一个线程
                 subTask2.fork();
             }
         }

    }*/

  static class AddTask extends RecursiveTask<Long>{
    int start,end;

    AddTask(int s,int e){
      start = s;
      end = e;
    }

    @Override
    protected Long compute() {

      if(end-start <= MAX_NUM) {
        long sum = 0L;
        for(int i = start;i<end;i++) {
          sum += nums[i];
        }
        return sum;
      }

      int middle = start + (end-start)/2;

      AddTask subTask1 = new AddTask(start, middle);
      AddTask subTask2 = new AddTask(middle, end);
      subTask1.fork(); 
      subTask2.fork();

      return subTask1.join() + subTask2.join(); //通过join()来获取子任务结果
    }
  }

  public static void main(String[] args) throws IOException {
    ForkJoinPool fjp = new ForkJoinPool();
    AddTask task = new AddTask(0,nums.length);
    fjp.execute(task);
    long result = task.join();
    System.out.println(result);

    //System.in.read();
  }
}
```

