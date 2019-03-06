#   java多线程系列--java多线程基础

[TOC]

## 相关概念

### 线程和进程有何不同？

线程是进程划分成的更小的运行单位。线程和进程最大的不同在于基本上**各进程是独立的，而各线程则不一定**，因为同一进程中的线程极有可能会相互影响。从另一角度来说，进程属于操作系统的范畴，主要是同一段时间内，可以同时执行一个以上的程序，而线程则是在同一程序内几乎同时执行一个以上的程序段。

### 何为多线程？

**多线程就是几乎同时执行多个线程**（一个处理器在某一个时间点上永远都只能是一个线程！即使这个处理器是多核的，除非有多个处理器才能实现多个线程同时运行。）。几乎同时是因为实际上多线程程序中的多个线程实际上是一个线程执行一会然后其他的线程再执行，并不是很多书籍所谓的同时执行。

## 线程的实现

### 继承Thread类

在`java.lang`包中定义, 继承Thread类必须重写`run()`方法

```java
class MyThread extends  Thread{
    private  String name;

    public MyThread(String name){
        this.name = name;
    }

    @Override
    public void run(){
        System.out.println("name:"+name+" 子线程ID:"+Thread.currentThread().getId());
    }
}
```

创建好了自己的线程类之后，就可以创建线程对象了，然后通过start()方法去启动线程。注意，不是调用run()方法启动线程，run()方法中只是定义需要执行的任务，如果调用run()方法，即相当于在主线程中执行run()方法，跟普通的方法调用没有任何区别，此时并不会创建一个新的线程来执行定义的任务。

```java
class MyThread extends  Thread{
    private  String name;

    public MyThread(String name){
        this.name = name;
    }

    @Override
    public void run(){
        System.out.println("name:"+name+" 子线程ID:"+Thread.currentThread().getId());
    }
}

public class Thread1Demo {
    @Test
    public void test(){
        System.out.println("主线程Id:"+Thread.currentThread().getId());
        MyThread myThread1 = new MyThread("thread1");
        myThread1.start();   //启动一个新线程

        MyThread myThread2 = new MyThread("thread2");
        myThread2.run();
    }
}
```

结果：

```java
主线程Id:1
name:thread2 子线程ID:1
name:thread1 子线程ID:10
```

从输出结果可以得出以下结论：

1）thread1和thread2的线程ID不同，thread2和主线程ID相同，说明通过run方法调用并不会创建新的线程，而是在主线程中直接运行run方法，跟普通的方法调用没有任何区别；

2）虽然thread1的start方法调用在thread2的run方法前面调用，但是先输出的是thread2的run方法调用的相关信息，说明新线程创建的过程不会阻塞主线程的后续执行。

 ### 实现Runnable接口

在Java中创建线程除了继承Thread类之外，还可以通过实现Runnable接口来实现类似的功能。实现Runnable接口必须重写其run方法。

```java
class MyRunnable implements  Runnable{
    private  String name;
    public MyRunnable(String name){
        this.name = name;
    }

    @Override
    public void run() {
        System.out.println("name:"+name+" 子线程ID:"+Thread.currentThread().getId());
    }
}

    @Test
    public void test2(){
        System.out.println("主线程Id:"+Thread.currentThread().getId());
        MyRunnable myRunnable = new MyRunnable("myRunable1");
        Thread thread = new Thread(myRunnable); //将任务交给thread去执行
        thread.start();
    }
```

 结果：

```java
主线程Id:1
name:myRunable1 子线程ID:10
```

Runnable的中文意思是“任务”，顾名思义，通过实现Runnable接口，我们定义了一个子任务，然后将子任务交由Thread去执行。注意，这种方式必须将Runnable作为Thread类的参数，然后通过Thread的start方法来创建一个新线程来执行该子任务。如果调用Runnable的run方法的话，是不会创建新线程的，这根普通的方法调用没有任何区别。

事实上，查看Thread类的实现源代码会发现Thread类是实现了Runnable接口的。

在Java中，这2种方式都可以用来创建线程去执行子任务，具体选择哪一种方式要看自己的需求。直接继承Thread类的话，可能比实现Runnable接口看起来更加简洁，但是由于Java只允许单继承，所以如果自定义类需要继承其他类，则只能选择实现Runnable接口。使⽤Runnable还有⼀ 个特点：使⽤Runnable实现的多线程的程序类可以更好的描述出程序共享的概念(并不是说Thread不能)

```java
class Mythread3 extends Thread{
    private  int ticket = 10; //一共10张票

    @Override
    public void run(){
        while (ticket>0){
            System.out.println("剩下票数："+ticket--);
        }
    }
}

public class Thread3Demo {

    public static void main(String[] args) {
        new Mythread3().start();
        new Mythread3().start();
        new Mythread3().start();
    }

}
```

此时启动三个线程实现卖票处理，结果变成了卖各自的票,共享写法如下：

```java
class Mythread3 extends Thread{
    private  int ticket = 10; //一共10张票

    @Override
    public void run(){
        while (ticket>0){
            System.out.println("剩下票数："+ticket--);
        }
    }
}

public class Thread3Demo {

    public static void main(String[] args) {
       Mythread3 mythread3 = new Mythread3();

       new Thread(mythread3).start();
       new Thread(mythread3).start();
       new Thread(mythread3).start();
    }

}
```

```java
剩下票数：10
剩下票数：9
剩下票数：8
剩下票数：7
剩下票数：6
剩下票数：5
剩下票数：4
剩下票数：3
剩下票数：2
剩下票数：1
```



使用Runnable实现共享

```java
class Mythread4 implements  Runnable{

    private  int ticket = 10; //一共10张票

    @Override
    public void run() {
        while (ticket>0){
            System.out.println("剩下票数："+ticket--);
        }
    }
}

public class Thread3Demo {

    public static void main(String[] args) {
        Mythread4 mythread4 = new Mythread4();
        Thread thread1 = new Thread(mythread4);
        thread1.start();

        Thread thread2 = new Thread(mythread4);
        thread2.start();
    }

}
```

结果：

```java
剩下票数：10
剩下票数：8
剩下票数：7
剩下票数：6
剩下票数：5
剩下票数：4
剩下票数：3
剩下票数：2
剩下票数：1
剩下票数：9
```

**总结：**

**实现Runnable接口比继承Thread类所具有的优势：**

- **适合多个相同的程序代码的线程去处理同一个资源**
- **可以避免java中的单继承的限制**
- **增加程序的健壮性，代码可以被多个线程共享，代码和数据独立**
- **线程池只能放入实现Runable或callable类线程，不能直接放入继承Thread的类 **

**提醒一下大家：main方法其实也是一个线程。在java中所有的线程都是同时启动的，至于什么时候，哪个先执行，完全看谁先得到CPU的资源。在java中，每次程序运行至少启动了2个线程，一个是main线程，一个是垃圾收集线程，**



 

### 实现Callable接口

从JDK1.5开始追加了新的开发包：java.uti.concurrent。这个开发包主要是进⾏⾼并发编程使⽤的，包含很多 在⾼并发操作中会使⽤的类。在这个包⾥定义有⼀个新的接⼝Callable

```java
@FunctionalInterface
public interface Callable<V> {
 /**
 * Computes a result, or throws an exception if unable to do so.
 *
 * @return computed result
 * @throws Exception if unable to compute a result
 */
 V call() throws Exception;
}
```

Runnable中的run()⽅法没有返回值，它的设计也遵循了主⽅法的设计原则：线程开始了就别回头。但是很多时候需要⼀些返回值，例如某些线程执⾏完成后可能带来⼀些返回结果，这种情况下就只能利⽤Callable来实现多线程

```java
class Mythread implements Callable{

    private int ticket = 10; //一共10张票
    @Override
    public Object call() throws Exception {

        while(ticket>0){
            System.out.println("剩下票数："+ticket--);
        }

        return "票卖完了，下次再来";
    }
}

public class Thread3Demo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        FutureTask<String> futureTask = new FutureTask<>(new Mythread());

        Thread thread1 = new Thread(futureTask);
        thread1.start();

        Thread thread2 = new Thread(futureTask);
        thread2.start();
        System.out.println(futureTask.get());
    }

}
```

结果：

```java
剩下票数：10
剩下票数：9
剩下票数：8
剩下票数：7
剩下票数：6
剩下票数：5
剩下票数：4
剩下票数：3
剩下票数：2
剩下票数：1
票卖完了，下次再来
```

来看Callable的继承树：

![选区_226.png](https://i.loli.net/2018/07/29/5b5d75d83fc3d.png)

注：UML图相关知识：https://my.oschina.net/jackieyeah/blog/224265

**不管何种情况。如果要想启动多线程只有Thread类中的start()⽅法。**



（二）：使用ExecutorService、Callable、Future实现有返回结果的多线程

 ExecutorService、Callable、Future这个对象实际上都是属于Executor框架中的功能类。想要详细了解Executor框架的可以访问[http://www.javaeye.com/topic/366591](https://link.jianshu.com?t=http://www.javaeye.com/topic/366591) ，这里面对该框架做了很详细的解释。返回结果的线程是在JDK1.5中引入的新特征，确实很实用，有了这种特征我就不需要再为了得到返回值而大费周折了，而且即便实现了也可能漏洞百出。

可**返回值的任务必须实现Callable接口，类似的，无返回值的任务必须Runnable接口**。执行Callable任务后，可以获取一个Future的对象，在该对象上调用get就可以获取到Callable任务返回的Object了，再结合线程池接口ExecutorService就可以实现传说中有返回结果的多线程了。下面提供了一个完整的有返回结果的多线程测试例子，在JDK1.8下验证过没问题可以直接使用。代码如下：

```java
class MyCallable implements Callable{
    private String taskNum;

    public MyCallable(String taskNum){
        this.taskNum = taskNum;
    }

    @Override
    public Object call() throws Exception {

        System.out.println(">>>"+taskNum+"任务启动");
        Date date1 = new Date();
        Thread.sleep(1000);
        Date date2 = new Date();
        long time = date2.getTime() - date1.getTime();
        System.out.println(">>>>"+taskNum+"任务终止");

        return taskNum + "任务返回运行结果,当前任务时间【" + time + "毫秒】";
    }
}

public class Thread2Demo {

    @Test
    public void test1() throws ExecutionException, InterruptedException {
        System.out.println("------程序开始运行-----");
        Date date1 = new Date();
        int taskSize = 5;
        //创建一个线程池
        ExecutorService executorService = Executors.newFixedThreadPool(taskSize);
        //创建有多个返回值的任务
        List<Future> list = new ArrayList<Future>();

        for (int i = 0; i < taskSize; ++i){
            Callable c = new MyCallable(i+" ");
            //执行任务并获取Future对象
            Future f = executorService.submit(c);
            list.add(f);
        }
        //关闭线程池
        executorService.shutdown();

        //获取所有并发任务的运行结果
        for (Future f: list){
            //从Future对象上获取任务的返回值，并输出到控制台
            System.out.println(">>>"+f.get().toString());
        }

        Date date2 = new Date();
        System.out.println("----程序结束运行----，程序运行时间【"
                + (date2.getTime() - date1.getTime()) + "毫秒】");
    }
}
```

 结果：

```java
------程序开始运行-----
>>>0 任务启动
>>>1 任务启动
>>>2 任务启动
>>>3 任务启动
>>>4 任务启动
>>>>0 任务终止
>>>0 任务返回运行结果,当前任务时间【1003毫秒】
>>>>1 任务终止
>>>1 任务返回运行结果,当前任务时间【1003毫秒】
>>>>2 任务终止
>>>2 任务返回运行结果,当前任务时间【1003毫秒】
>>>>3 任务终止
>>>3 任务返回运行结果,当前任务时间【1003毫秒】
>>>>4 任务终止
>>>4 任务返回运行结果,当前任务时间【1002毫秒】
----程序结束运行----，程序运行时间【1016毫秒】
```

代码说明：
 上述代码中Executors类，提供了一系列工厂方法用于创先线程池，返回的线程池都实现了ExecutorService接口。

-  public static ExecutorService newFixedThreadPool(int nThreads)

   创建固定数目线程的线程池。

- public static ExecutorService newCachedThreadPool()

  创建一个可缓存的线程池，调用execute 将重用以前构造的线程（如果线程可用）。如果现有线程没有可用的，则创建一个新线程并添加到池中。终止并从缓存中移除那些已有 60 秒钟未被使用的线程。

- public static ExecutorService newSingleThreadExecutor()

   创建一个单线程化的Executor。

- public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize)

  创建一个支持定时及周期性的任务执行的线程池，多数情况下可用来替代Timer类。

ExecutoreService提供了submit()方法，传递一个Callable，或Runnable，返回Future。如果Executor后台线程池还没有完成Callable的计算，这调用返回Future对象的get()方法，会阻塞直到计算完成

## 线程的状态

在正式学习Thread类中的具体方法之前，我们先来了解一下线程有哪些状态，这个将会有助于后面对Thread类中的方法的理解。

- 创建（new）状态: 准备好了一个多线程的对象 new Thread()
- 就绪（runnable）状态: 调用了`start()`方法, 等待CPU进行调度
- 运行（running）状态: 执行`run()`方法
- 阻塞（blocked）状态: 暂时停止执行, 可能将资源交给其它线程使用
- 终止（dead）状态: 线程销毁

  当需要新起一个线程来执行某个子任务时，就创建了一个线程。但是线程创建之后，不会立即进入就绪状态，因为线程的运行需要一些条件，只有线程运行需要的所有条件满足了，才进入就绪状态。

当线程进入就绪状态后，不代表立刻就能获取CPU执行时间，也许此时CPU正在执行其他的事情，因此它要等待。当得到CPU执行时间之后，线程便真正进入运行状态。

线程在运行状态过程中，可能有多个原因导致当前线程不继续运行下去，比如用户主动让线程睡眠（睡眠一定的时间之后再重新执行）、用户主动让线程等待，或者被同步块给阻塞，此时就对应着多个状态：time waiting（睡眠或等待一定的事件）、waiting（等待被唤醒）、blocked（阻塞）。

当由于突然中断或者子任务执行完毕，线程就会被消亡。

下面这副图描述了线程从创建到消亡之间的状态：

 ![选区_225.png](https://i.loli.net/2018/07/29/5b5d6aa3ae690.png)

注意：start()方法的调用后并不是立即执行多线程代码，而是使得该线程变为可运行态（Runnable），什么时候运行是由操作系统决定的，**但是一个线程的start方法重复调用的话，会报出java.lang.IllegalThreadStateException异常**。

在有些教程上将blocked、waiting、time waiting统称为阻塞状态，这个也是可以的，只不过这里我想将线程的状态和Java中的方法调用联系起来，所以将waiting和time waiting两个状态分离出来。

注:sleep和wait的区别:

- `sleep`是`Thread`类的方法,`wait`是`Object`类中定义的方法.
- `Thread.sleep`不会导致锁行为的改变, 如果当前线程是拥有锁的, 那么`Thread.sleep`不会让线程释放锁.
- `Thread.sleep`和`Object.wait`都会暂停当前的线程. OS会将执行时间分配给其它线程. 区别是, 调用`wait`后, 需要别的线程执行`notify/notifyAll`才能够重新获得CPU执行时间.

## 上下文切换

对于单核CPU来说，CPU在一个时刻只能运行一个线程，当在运行一个线程的过程中转去运行另外一个线程，这个叫做线程上下文切换（对于进程也是类似）。

由于可能当前线程的任务并没有执行完毕，所以在切换时需要保存线程的运行状态，以便下次重新切换回来时能够继续切换之前的状态运行。举个简单的例子：比如一个线程A正在读取一个文件的内容，正读到文件的一半，此时需要暂停线程A，转去执行线程B，当再次切换回来执行线程A的时候，我们不希望线程A又从文件的开头来读取。

因此需要记录线程A的运行状态，那么会记录哪些数据呢？因为下次恢复时需要知道在这之前当前线程已经执行到哪条指令了，所以需要记录程序计数器的值，另外比如说线程正在进行某个计算的时候被挂起了，那么下次继续执行的时候需要知道之前挂起时变量的值时多少，因此需要记录CPU寄存器的状态。所以一般来说，线程上下文切换过程中会记录程序计数器、CPU寄存器状态等数据。

说简单点的：对于线程的上下文切换实际上就是 **存储和恢复CPU状态的过程，它使得线程执行能够从中断点恢复执行**。

虽然多线程可以使得任务执行的效率得到提升，但是由于在线程切换时同样会带来一定的开销代价，并且多个线程会导致系统资源占用的增加，所以在进行多线程编程时要注意这些因素。

## 线程的常用方法

  ![选区_224.png](https://i.loli.net/2018/07/29/5b5d55e7e633a.png)

currentThread()方法

```java
class Mythread6 implements Runnable{

    @Override
    public void run() {
        for (int i = 0; i < 3 ; i++) {
            System.out.println("当前线程："+Thread.currentThread().getName()+" ,i ="+i);
        }
    }
}

public class Thread3Demo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

       Mythread6 mythread6 = new Mythread6();

       new Thread(mythread6).start();    //没有设置名字
       new Thread(mythread6).start();
       new Thread(mythread6,"huangjia").start();   //有设置名字
    }

}
```

结果：

```java
当前线程：Thread-0 ,i =0
当前线程：Thread-1 ,i =0
当前线程：Thread-0 ,i =1
当前线程：Thread-0 ,i =2
当前线程：Thread-1 ,i =1
当前线程：Thread-1 ,i =2
当前线程：huangjia ,i =0
当前线程：huangjia ,i =1
当前线程：huangjia ,i =2
```

通过上述代码发现，如果没有设置线程名字，则会⾃动分配⼀个线程名字。需要主要的是，线程名字如果要 设置请避免重复，同时中间不要修改。

 yield()方法

调用yield方法会让当前线程交出CPU权限，让CPU去执行其他的线程。它跟sleep方法类似，同样不会释放锁。但是yield不能控制具体的交出CPU的时间，另外，yield方法只能让拥有相同优先级的线程有获取CPU执行时间的机会。

注意，调用yield方法并不会让线程进入阻塞状态，而是让线程重回就绪状态，它只需要等待重新获取CPU执行时间，这一点是和sleep方法不一样的。

 ```java
 class MyThread2  extends Thread{
    @Override
    public void run() {
        long beginTime=System.currentTimeMillis();
        int count=0;
        for (int i=0;i<50000000;i++){
            count=count+(i+1);
            Thread.yield();
        }
        long endTime=System.currentTimeMillis();
        System.out.println("用时："+(endTime-beginTime)+" 毫秒！");
    }
}

  public static void main(String[] args) {
        MyThread2 myThread2 = new MyThread2();
        myThread2.start();
    }
 ```

结果：

```java
用时：19798 毫秒！
```

如果将Thread.yield()方法注释掉，执行结果如下：

```java
用时：25 毫秒！
```

join()方法

join是Thread类的一个方法，启动线程后直接调用，即join()的作用是：“等待该线程终止”，这里需要理解的就是该线程是指的主线程等待子线程的终止。也就是在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行。

在很多情况下，主线程生成并启动子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，也就是主线程需要等待子线程执行完成之后再结束，这个时候就要用到join()方法了。

不加join

```java
class Thread1 extends Thread{
	private String name;
    public Thread1(String name) {
       this.name=name;
    }
	public void run() {
		System.out.println(Thread.currentThread().getName() + " 线程运行开始!");
        for (int i = 0; i < 5; i++) {
            System.out.println("子线程"+name + "运行 : " + i);
            try {
                sleep((int) Math.random() * 10);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
        System.out.println(Thread.currentThread().getName() + " 线程运行结束!");
	}
}
 
public class Main {
 
	public static void main(String[] args) {
		System.out.println(Thread.currentThread().getName()+"主线程运行开始!");
		Thread1 mTh1=new Thread1("A");
		Thread1 mTh2=new Thread1("B");
		mTh1.start();
		mTh2.start();
		System.out.println(Thread.currentThread().getName()+ "主线程运行结束!");
 
	}
 
}
```

结果：

```java
main主线程运行开始!
main主线程运行结束!
B 线程运行开始!
子线程B运行 : 0
A 线程运行开始!
子线程A运行 : 0
子线程B运行 : 1
子线程A运行 : 1
子线程A运行 : 2
子线程A运行 : 3
子线程A运行 : 4
A 线程运行结束!
子线程B运行 : 2
子线程B运行 : 3
子线程B运行 : 4
B 线程运行结束!
```

加上join方法：

```java

public class Main {
 
	public static void main(String[] args) {
		System.out.println(Thread.currentThread().getName()+"主线程运行开始!");
		Thread1 mTh1=new Thread1("A");
		Thread1 mTh2=new Thread1("B");
		mTh1.start();
		mTh2.start();
		try {
			mTh1.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		try {
			mTh2.join();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
		System.out.println(Thread.currentThread().getName()+ "主线程运行结束!");
 
	}
 
```

结果：

```java
main主线程运行开始!
A 线程运行开始!
B 线程运行开始!
子线程A运行 : 0
子线程B运行 : 0
子线程A运行 : 1
子线程B运行 : 1
子线程A运行 : 2
子线程B运行 : 2
子线程A运行 : 3
子线程B运行 : 3
子线程A运行 : 4
子线程B运行 : 4
A 线程运行结束!
B 线程运行结束!
main主线程运行结束!
```
为啥mth1.join阻塞的是主线程？

mth1.join()方法是主线程执行， 线程执行完毕以后会有一个唤醒的操作，只是我们不需要关心。

https://www.jianshu.com/p/fc51be7e5bc0

```java
public final synchronized void join(long millis)
throws InterruptedException {
    long base = System.currentTimeMillis();
    long now = 0;

    if (millis < 0) {
        throw new IllegalArgumentException("timeout value is negative");
    }

    if (millis == 0) {
        while (isAlive()) { //isAlive()方法是mth1对象的方法
            wait(0);    //主线程运行到这阻塞， 
        }
    } else {
        while (isAlive()) {
            long delay = millis - now;
            if (delay <= 0) {
                break;
            }
            wait(delay);
            now = System.currentTimeMillis() - base;
        }
    }
}
```

>  面试题： Java中如何让多线程按照自己指定的顺序执行？

- 最简单的方法就是通过Thread.join来实现了。

## 线程优先级

每个线程都具有各自的优先级，线程的优先级可以在程序中表明该线程的重要性，如果有很多线程处于就绪状态，系统会根据优先级来决定首先使哪个线程进入运行状态。但这个并不意味着低 优先级的线程得不到运行，而只是它运行的几率比较小，如垃圾回收机制线程的优先级就比较低。所以很多垃圾得不到及时的回收处理。

线程优先级具有继承特性比如A线程启动B线程，则B线程的优先级和A是一样的。

线程优先级具有随机性也就是说线程优先级高的不一定每一次都先执行完。

Thread类中包含的成员变量代表了线程的某些优先级。如**Thread.MIN_PRIORITY（常数1）**，**Thread.NORM_PRIORITY（常数5）**, **Thread.MAX_PRIORITY（常数10）**。其中每个线程的优先级都在**Thread.MIN_PRIORITY（常数1）** 到**Thread.MAX_PRIORITY（常数10）** 之间，在默认情况下优先级都是**Thread.NORM_PRIORITY（常数5）**。

```java
class Mythread3 extends Thread{
    @Override
    public void run() {
        System.out.println("MyThread1 run priority=" + this.getPriority());
        MyThread4 thread4 = new MyThread4();
        thread4.start();
    }

}

class MyThread4 extends Thread {
    @Override
    public void run() {
        System.out.println("MyThread2 run priority=" + this.getPriority());
    }
}


public class Thread3Demo {

    public static void main(String[] args) throws InterruptedException {
        System.out.println("main thread begin priority="
                + Thread.currentThread().getPriority());
        Thread.currentThread().setPriority(6);
        System.out.println("main thread end   priority="
                + Thread.currentThread().getPriority());
        Mythread3 thread3 = new Mythread3();
        thread3.start();


    }

}
```

结果：

```java
main thread begin priority=5
main thread end   priority=6
MyThread1 run priority=6
MyThread2 run priority=6
```

## Java多线程分类

### 多线程分类

- 用户线程：运行在前台，执行具体的任务，如程序的主线程、连接网络的子线程等都是用户线程
- 守护线程：运行在后台，为其他前台线程服务.也可以说守护线程是JVM中非守护线程的 **“佣人”**。
- 特点：一旦所有用户线程都结束运行，守护线程会随JVM一起结束工作
- 应用：数据库连接池中的检测线程，JVM虚拟机启动后的检测线程
- 最常见的守护线程：垃圾回收线程

### 如何设置守护线程

> 可以通过调用Thead类的setDaemon(true)方法设置当前的线程为守护线程

 注意事项：

- setDaemon(true)必须在start（）方法前执行，否则会抛出IllegalThreadStateException异常 
- 在守护线程中产生的新线程也是守护线程
- 不是所有的任务都可以分配给守护线程来执行，比如读写操作或者计算逻辑

 ```java
class Mythread3 extends Thread{
    private int i = 0;

    @Override
    public void run() {
        try {
            while (true) {
                i++;
                System.out.println("i=" + (i));
                Thread.sleep(100);
            }
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
    }

}

public class Thread3Demo {

    public static void main(String[] args) throws InterruptedException {
       Mythread3 mythread3 = new Mythread3();
       mythread3.setDaemon(true);
       mythread3.start();
       Thread.sleep(5000);
        System.out.println("我离开了thread对象再也不打印了，也就是停止了");
    }

}
 ```

结果：

```java
i=44
i=45
i=46
i=47
i=48
i=49
i=50
我离开了thread对象再也不打印了，也就是停止了
```



 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

 

   