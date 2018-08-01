# java多线程系列--Executor框架和线程池

[TOC]

## 使用线程池的好处

**线程池**提供了一种限制和管理资源（包括执行一个任务）。 每个**线程池**还维护一些基本统计信息，例如已完成任务的数量。
**使用线程池的好处**：

-  **降低资源消耗**。通过重复利用已创建的线程降低线程创建和销毁造成的消耗。
-  **提高响应速度**。当任务到达时，任务可以不需要的等到线程创建就能立即执行。
-  **提高线程的可管理性**。线程是稀缺资源，如果无限制的创建，不仅会消耗系统资源，还会降低系统的稳定性，使用线程池可以进行统一的分配，调优和监控。

    ## Executor框架

其内部使用了线程池机制，它在java.util.cocurrent 包下，通过该框架来控制线程的启动、执行和关闭，可以简化并发编程的操作。因此，在Java 5之后，通过Executor来启动线程比使用Thread的start方法更好，除了更易管理，效率更好（用线程池实现，节约开销）外，还有关键的一点：有助于避免this逃逸问题——如果我们在构造器中启动一个线程，因为另一个任务可能会在构造器结束之前开始执行，此时可能会访问到初始化了一半的对象。

![选区_240.png](https://i.loli.net/2018/07/30/5b5f2e5c133c3.png)

### Executor接口

```java
public interface Executor {
    void execute(Runnable command);
}
```

Executor接口是Executor框架中最基础的部分，定义了一个用于执行Runnable的execute方法，它没有实现类只有另一个重要的子接口ExecutorService。

### ExecutorService接口

```java
//继承自Executor接口
  public interface ExecutorService extends Executor {
      /**
       * 关闭方法，调用后执行之前提交的任务，不再接受新的任务
       */
      void shutdown();
      /**
       * 从语义上可以看出是立即停止的意思，将暂停所有等待处理的任务并返回这些任务的列表
       */
     List<Runnable> shutdownNow();
     /**
      * 判断执行器是否已经关闭
      */
     boolean isShutdown();
     /**
      * 关闭后所有任务是否都已完成
      */
     boolean isTerminated();
     /**
      * 中断
      */
    boolean awaitTermination(long timeout, TimeUnit unit)
         throws InterruptedException;
     /**
      * 提交一个Callable任务
      */
     <T> Future<T> submit(Callable<T> task);
     /**
      * 提交一个Runable任务，result要返回的结果
      */
     <T> Future<T> submit(Runnable task, T result);
     /**
      * 提交一个任务
      */
     Future<?> submit(Runnable task);
 }
```

ExecutorService接口继承自Executor接口，定义了**终止、提交、执行任务**、跟踪任务返回结果等方法

- **execute（Runnable command）：履行Ruannable类型的任务,**
- **submit（task）：可用来提交Callable或Runnable任务，并返回代表此任务的Future对象**

#### Runnable、Callable、Future接口

**Runnable接口**

```java
// 实现Runnable接口的类将被Thread执行，表示一个基本的任务
  public interface Runnable {
      // run方法就是它所有的内容，就是实际执行的任务
      public abstract void run();
  }
```

**Callable接口:**与Runnable接口的区别在于**它接收泛型**，同时它执行任务后**带有返回内容**

```java
// Callable同样是任务，与Runnable接口的区别在于它接收泛型，同时它执行任务后带有返回内容
  public interface Callable<V> {
      // 相对于run方法的带有返回值的call方法
      V call() throws Exception;
}
```

**Future接口**

```java
// Future代表异步任务的执行结果
  public interface Future<V> {
  
      /**
       * 尝试取消一个任务，如果这个任务不能被取消（通常是因为已经执行完了），返回false，否则返回true。
       */
      boolean cancel(boolean mayInterruptIfRunning);
  
      /**
      * 返回代表的任务是否在完成之前被取消了
      */
     boolean isCancelled();
 
     /**
      * 如果任务已经完成，返回true
      */
    boolean isDone();
 
     /**
      * 获取异步任务的执行结果（如果任务没执行完将等待）
      */
    V get() throws InterruptedException, ExecutionException;
 
     /**
      * 获取异步任务的执行结果（有最常等待时间的限制）
      *
      *  timeout表示等待的时间，unit是它时间单位
      */
     V get(long timeout, TimeUnit unit)
         throws InterruptedException, ExecutionException, TimeoutException;
 }
```

在Future接口中声明了5个方法，下面依次解释每个方法的作用：

- cancel方法用来取消任务，如果取消任务成功则返回true，如果取消任务失败则返回false。参数mayInterruptIfRunning表示是否允许取消正在执行却没有完的任务，如果设置true，则表示可以取消正在执行的任务。如果任务已经完成，则无论mayInterruptIfRunning为true还是false，此方法肯定返回false；如果任务还没有执行，则无论mayInterruptIfRunning为true还是false，肯定返回true；如果任务正在执行，若mayInterruptIfRunning设置为true，则返回true，若mayInterruptIfRunning设置为false，则返回false。
- isCancelled方法表示任务是否被取消成功，如果在任务正常完成前被取消成功，则返回 true。
- isDone方法表示任务是否已经完成，若任务完成，则返回true。
-  get方法用来获取执行结果，这个方法会产生阻塞，会一直等到任务执行完毕才返回。
- get(long timeout, TimeUnit unit)用来获取执行结果，如果在指定时间内，还没获取到结果，就直接返回null。

总结：Future提供了三种功能：

1. 判断任务是否完成

2. 能够中断任务　
3. 能够获取任务执行结果

##### FutureTask类

通常使用FutureTask来处理我们的任务。我们以**AbstractExecutorService**接口中的一个submit方法为例子来看看源代码：

```java
public Future<?> submit(Runnable task) {
        if (task == null) throw new NullPointerException();
        RunnableFuture<Void> ftask = newTaskFor(task, null);
        execute(ftask);   //调用execute方法执行任务
        return ftask;
    }
```

上面方法调用的newTaskFor方法返回了一个FutureTask对象。

```java
protected <T> RunnableFuture<T> newTaskFor(Runnable runnable, T value) {
        return new FutureTask<T>(runnable, value);
    }
```

事实上，FutureTask是Future接口的唯一实现类，**FutureTask类同时又实现了Runnable接口，所以可以直接提交给Executor执行**。

```java
//FutureTask提供了2个构造器：
public FutureTask(Callable<V> callable) {
}
public FutureTask(Runnable runnable, V result) {
}
```

**不直接构造Future对象，也可以使用ExecutorService.submit方法来获得Future对象，submit方法即支持以 Callable接口类型，也支持Runnable接口作为参数**，具有很大的灵活性。使用示例如下：

```java
ExecutorService executor = Executors.newSingleThreadExecutor();   
FutureTask<String> future =　executor.submit(   
   new Callable<String>() {//使用Callable接口作为构造参数   
       public String call() {   
      //真正的任务在这里执行，这里的返回值类型为String，可以为任意类型   
   }});   
//在这里可以做别的任何事情   
```

### ThreadPoolExecutor类

ThreadPoolExecutor是线程池的实现类：

```java
public ThreadPoolExecutor(int corePoolSize,  
                              int maximumPoolSize,  
                              long keepAliveTime,  
                              TimeUnit unit,  
                              BlockingQueue<Runnable> workQueue,  
                              ThreadFactory threadFactory,  
                              RejectedExecutionHandler handler)   
```

（1）corePoolSize（线程池的基本大小）：当提交一个任务到线程池时，线程会创建一个线程来执行任务，**即使其他的线程空闲也会创建线程**，等到需要执行的任务数大于线程池基本大小corePoolSize时就不再创建。

（2）maximumPoolSize（线程池最大大小）：**线程池允许最大线程数**。如果阻塞队列**满了，并且已经创建的线程数小于最大线程数，则线程池会再创建新的线程执行**。因为**线程池执行任务时是线程池基本大小满了，后续任务进入阻塞队列，阻塞队列满了，再创建线程**。

（3）keepAliveTime（线程活动保持时间）：**空闲线程的保持存活时间。**

（4）TimeUnit（线程活动保持时间的单位）：
​        TimeUnit.DAYS; //天
        TimeUnit.SECONDS; //秒
        TimeUnit.MILLISECONDS; //毫秒
        TimeUnit.NANOSECONDS; //纳秒

（5）workQueue（任务队列）：**用于保存等待执行的任务的阻塞队列**。一个阻塞队列，用来存储等待执行的任务：**数组，链表，不存元素的阻塞队列**
​      5.1）ArrayBlockingQueue;**数组**结构的**有界**阻塞队列，先进先出FIFO
      5.2）LinkedBlockingQueue;**链表**结构的**无界**阻塞队列。先进先出FIFO排序元素
​      5.3）SynchronousQueue;**不存储元素**的阻塞队列，就是**每次插入操作必须等到另一个线程调用移除操作**

（6）threadFactory：用于设置创建线程的工厂，可以通过线程工厂给每个创建出来的线程设置更有意义的名字

（7）handler（饱和策略）：表示当拒绝处理任务时的策略。**当队列和线程池都满了，说明线程池处于饱和状态，那么必须采取一种策略处理提交的新任务。**这个策略**默认情况下是AbortPolicy**，表示无法处理新任务时抛出异常。
​     ThreadPoolExecutor.**AbortPolicy:丢弃任务并抛出RejectedExecutionException异常。**
​     ThreadPoolExecutor.**DiscardPolicy：也是丢弃任务，但是不抛出异常**。
​     ThreadPoolExecutor.DiscardOldestPolicy：丢弃队列最前面的任务，然后重新尝试执行任务（重复此过程）
​     ThreadPoolExecutor.CallerRunsPolicy：由调用线程处理该任务

#### Executor工厂类

通过Executors提供四种线程池，newFixedThreadPool、newCachedThreadPool、newSingleThreadExecutor、newScheduledThreadPool。

- 1.public static ExecutorService newFixedThreadPool(int nThreads) 

  创建固定数目线程的线程池。

- 2.public static ExecutorService newCachedThreadPool() 

  创建一个可缓存的线程池，调用execute将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。

- 3.public static ExecutorService newSingleThreadExecutor() 

  创建一个单线程化的Executor。线程池中只有一个线程，依次去执行任务

- 4.public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) 

  创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。

我们尽量优先使用Executors提供的静态方法来创建线程池，如果Executors提供的方法无法满足要求，再自己通过ThreadPoolExecutor类来创建线程池

下面是这三个静态方法的具体实现：

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}

public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}

public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}

```

从它们的具体实现来看，它们实际上也是调用了ThreadPoolExecutor，只不过参数都已配置好了。
newFixedThreadPool创建的**线程池corePoolSize和maximumPoolSize值是相等的(n,n)**，它使用的**LinkedBlockingQueue**；

newSingleThreadExecutor**将corePoolSize和maximumPoolSize都设置为1(1,1)**，也使用的**LinkedBlockingQueue**；

newCachedThreadPool将**corePoolSize设置为0，将maximumPoolSize设置为Integer.MAX_VALUE，**使用的**SynchronousQueue**，也就是说来了任务就创建线程运行，当线程空闲超过60秒，就销毁线程。



**详细分析三个线程池的执行过程**

- newFixedThreadPool:（固定线程池）

```java
public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

线程池corePoolSize和maximumPoolSize值是相等的(n,n),把keepAliveTime设置0L，意味着多余的空闲线程会被立即终止。

**newFixedThreadPool的execute方法执行过程：**

​	1，如果当前运行线程数少于corePoolSize，则创建新线程来执行任务（**优先满足核心池**）

​	2，当前运行线程数等于corePoolSize时，将任务加入LinkedBlockingQueue链式阻塞队列（**核心池满了在			     进入队列**）

​	3，当线程池的任务完成之后，循环反复从LinkedBlockingQueue**队列中获取任务来执行**

- newSingleThreadExecutor：(单线程执行器)

```java
public static ExecutorService newSingleThreadExecutor() {
    return new FinalizableDelegatedExecutorService
        (new ThreadPoolExecutor(1, 1,
                                0L, TimeUnit.MILLISECONDS,
                                new LinkedBlockingQueue<Runnable>()));
}
```

**newSingleThreadExecutor的execute方法执行过程如下：**

 	 1，当前运行的线程数少于corePoolSize（即当前线程池中午运行的线程），则创建一个新的线程来执行任务

​	  2，当线程池中有一个运行的线程时，将任务加入阻塞队列

​	  3，当线程完成任务时，会无限反复从链式阻塞队列中获取任务来执行

- newCachedThreadPool:可缓存线程池

```java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,
                                  new SynchronousQueue<Runnable>());
}
```

corePoolSize设置0，即核心池是空，maxmumPoolSize设置为Integer.MAX_VALUE,即maxmumPool是无界的。keepAliveTime设置60L,当空闲线程等待新任务最长时间是60s，超过60s就终止

**总结三个线程池的特点：**

  1、newFixedThreadPool创建一个**指定工作线程数量**的线程池。每当提交一个任务就创建一个工作线程，**如果工作线程数量达到线程池初始的最大数corePoolSize，则将提交的任务存入到池队列中**。

  2、newCachedThreadPool创建一个**可缓存的线程池**。这种类型的线程池特点是： 

​	1).**工作线程的创建数量几乎没有限制(**其实也有限制的,数目为Interger. MAX_VALUE), 这样可灵活的往线程池中添加线程。

​	 2).**如果长时间没有往线程池中提交任务，即如果工作线程空闲了指定的时间(默认为1分钟)，则该工作线程将自动终止。**终止后，如果你又提交了新的任务，则线程池重新创建一个工作线程。

  3、newSingleThreadExecutor创建一个**单线程化的Executor**，即只创建唯一的工作者线程来执行任务，如果这个线程异常结束，会有另一个取代它，保证顺序执行(我觉得这点是它的特色)。单工作线程最大的特点是**可保证顺序地执行各个任务**，并且在任意给定的时间不会有多个线程是活动的 

#### 线程池本身的状态

```java
volatile int runState;   
static final int RUNNING = 0;   //运行状态
static final int SHUTDOWN = 1;   //关闭状态
static final int STOP = 2;       //停止
static final int TERMINATED = 3; //终止，终结
```

 1，当创建线程池后，**初始时，线程池处于RUNNING状态**；

 2，**如果调用了shutdown()方法，则线程池处于SHUTDOWN状态**，此时线程池不能够接受新的任务，它会等待所有任务执行完毕，最后终止；

 3，**如果调用了shutdownNow()方法，则线程池处于STOP状态**，此时线程池不能接受新的任务，并且会去尝试终止正在执行的任务，返回没有执行的任务列表；

 4，当**线程池处于SHUTDOWN或STOP状态，并且所有工作线程已经销毁，任务缓存队列已经清空或执行结束后，线程池被设置为TERMINATED状态**。

###  ScheduledThreadPoolExecutor类

ScheduledThreadPoolExecutor主要用来在给定的延迟后运行任务，或者定期执行任务。
ScheduledThreadPoolExecutor使用的任务队列DelayQueue封装了一个PriorityQueue，PriorityQueue会对队列中的任务进行排序，执行所需时间短的放在前面先被执行(ScheduledFutureTask的time变量小的先执行)，如果执行所需时间相同则先提交的任务将被先执行(ScheduledFutureTask的squenceNumber变量小的先执行)

 **ScheduledThreadPoolExecutor运行机制**

![选区_246.png](https://i.loli.net/2018/08/01/5b61b4a31127b.png)

 

**ScheduledThreadPoolExecutor的执行主要分为两大部分：** 

1. 当调用ScheduledThreadPoolExecutor的 **scheduleAtFixedRate()** 方法或者**scheduleWirhFixedDelay()** 方法时，会向ScheduledThreadPoolExecutor的 **DelayQueue** 添加一个实现了 **RunnableScheduledFutur** 接口的 **ScheduledFutureTask** 。
2. 线程池中的线程从DelayQueue中获取ScheduledFutureTask，然后执行任务。

**ScheduledThreadPoolExecutor为了实现周期性的执行任务，对ThreadPoolExecutor做了如下修改：**

- 使用 **DelayQueue** 作为任务队列；
- 获取任务的方不同
- 执行周期任务后，增加了额外的处理

**ScheduledThreadPoolExecutor执行周期任务的步骤**

![选区_247.png](https://i.loli.net/2018/08/01/5b61b63f01cbe.png)

1. 线程1从DelayQueue中获取已到期的ScheduledFutureTask（DelayQueue.take()）。到期任务是指ScheduledFutureTask的time大于等于当前系统的时间；
2. 线程1执行这个ScheduledFutureTask；
3. 线程1修改ScheduledFutureTask的time变量为下次将要被执行的时间；
4. 线程1把这个修改time之后的ScheduledFutureTask放回DelayQueue中（DelayQueue.add())。

 

 

 

 

 



 

 

 

 

 

 

 