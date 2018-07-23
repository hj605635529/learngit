# java定时任务

[TOC]

在实际的开发过程中，如果业务逻辑相对简单，仅执行一些简单的定时任务，不需要相对复杂的控制。可以使用Java-JDK中自带的定时任务Timer来完成，提高开发效率。一个完整的定时任务需要**Timer（驱动者）、TimerTask（执行者）**两个类完成。

## Timer

​        关于**Timer**，JDK是这样描述的：**一种工具，线程用其安排以后在后台线程中执行的任务。可安排任务执行一次，或者定期重复执行。** 就是驱动TimerTask执行相应的任务，也就是说，到点了，告诉TimerTask：嘿，伙计，你小子该干活儿了。

这个类提供了4个构造方法，每个方法都会创建一个**新计时器，**并启动**计时器线程。**Timer类可以保证**多个线程共享单个Timer对象**而无需进行外部同步，所以**Timer类是线程安全的。*。由于每个Timer对象对应的是**单个后台进程**，这个后台进程用**于顺序执行所有的定时任务。**如果个别定时任务**执行任务消耗时间过长**，它就会**“独占”**计时器的任务执行线程，后面的线程必须等它执行完才继续执行后面的任务。这会造成后面的任务并非根据设置的时间执行，最终导致**延迟后续任务的执行**。这是一个弊端，

​        下图是JDK中关于Timer的描述，大家可以看看里面的方法介绍：

![选区_190.png](https://i.loli.net/2018/07/23/5b55f177e0a47.png)



### TimerTask

​        **TimerTask**是一个抽象类**，**官方给出的解释：**由 Timer 安排为一次执行或重复执行的任务。**也就是说**，TimerTask是具体的任务执行者，要想添加执行任务，需要继承TimerTask并重写run()方法** 

![选区_191.png](https://i.loli.net/2018/07/23/5b55f21e2d666.png)

### Time方法的应用

１．

```java
class MyTask1 extends  TimerTask{

    @Override
    public void run(){
        System.out.println("mytask1执行了,执行时间.."+new Date());
        try {
            sleep(6000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("休眠时间到了，执行时间..."+new Date());
    }
}

class MyTask2 extends  TimerTask{

    @Override
    public void run(){
        System.out.println("mytask2执行了,执行时间.."+new Date());
    }
}

public class TimerDemo {

    public static void main(String[] args) {
        Timer timer = new Timer();

        timer.schedule(new MyTask1(),3000);

        timer.schedule(new MyTask2(),5000);
    }

}
```

结果：

```java
mytask1执行了,执行时间..Mon Jul 23 23:44:31 CST 2018
休眠时间到了，执行时间...Mon Jul 23 23:44:37 CST 2018
mytask2执行了,执行时间..Mon Jul 23 23:44:37 CST 2018
```

从结果中我们可以看到mytask1与mytask2执行的时间相隔了6秒，但是我们在实例中只是相隔了2秒，由于所有任务都是由同一个线程来调度，**因此所有任务都是串行执行的**，同一时间只能有一个任务在执行，前一个任务的延迟或异常都将会影响到之后的任务。

2.

```java
class MyTask1 extends  TimerTask{

    @Override
    public void run(){
        System.out.println("mytask1执行了,执行时间.."+new Date());
        try {
            sleep(1000*15);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("休眠时间到了，执行时间..."+new Date());
    }
}

class MyTask2 extends  TimerTask{

    @Override
    public void run(){
        System.out.println("mytask2执行了,执行时间.."+new Date());
    }
}

public class TimerDemo {

    public static void main(String[] args) {
        Timer timer = new Timer();

        //3秒后开始启动
        timer.scheduleAtFixedRate(new MyTask1(),3000,1000*10);

        //5秒后开始启动
        timer.scheduleAtFixedRate(new MyTask2(),5000,1000*10);
    }

}
```

结果：

```java
mytask1执行了,执行时间..Tue Jul 24 00:07:21 CST 2018
休眠时间到了，执行时间...Tue Jul 24 00:07:36 CST 2018
mytask2执行了,执行时间..Tue Jul 24 00:07:36 CST 2018
mytask1执行了,执行时间..Tue Jul 24 00:07:36 CST 2018
休眠时间到了，执行时间...Tue Jul 24 00:07:51 CST 2018
mytask2执行了,执行时间..Tue Jul 24 00:07:51 CST 2018
```

从结果中我们看出仍能有延时，而不是象网上说的它的**task执行任务时间=第一次执行时间+n\*period**

3.

```java
class MyTask1 extends  TimerTask{

    @Override
    public void run(){
        System.out.println("mytask1执行了,执行时间.."+new Date());
        try {
            sleep(6000);
            throw new RuntimeException();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("休眠时间到了，执行时间..."+new Date());
    }
}

class MyTask2 extends  TimerTask{

    @Override
    public void run(){
        System.out.println("mytask2执行了,执行时间.."+new Date());
    }
}

public class TimerDemo {

    public static void main(String[] args) {
        Timer timer = new Timer();

        //3秒后开始启动
        timer.scheduleAtFixedRate(new MyTask1(),3000,1000*10);

        //5秒后开始启动
        timer.scheduleAtFixedRate(new MyTask2(),5000,1000*10);
    }

}
```

结果：

```java
mytask1执行了,执行时间..Mon Jul 23 23:58:04 CST 2018
Exception in thread "Timer-0" java.lang.RuntimeException
	at com.hj.MyTask1.run(TimerDemo.java:23)
	at java.util.TimerThread.mainLoop(Timer.java:555)
	at java.util.TimerThread.run(Timer.java:505)
```

从结果中我们能看出**Timer抛出异常时，导致Timer终止所有定时任务**

## ScheduledThreadPoolExecutor

Timer内部是**单一线程**，而**ScheduledThreadPoolExecutor**内部是个**线程池**，所以，你懂得。支持多个任务并发执行简直so easy。

**1、使用ScheduledExecutorService代替Timer，解决task任务执行延迟问题**

```java
class MyTask1 implements Runnable{

    @Override
    public void run(){
        System.out.println("mytask1执行了,执行时间.."+new Date());
        try {
            sleep(5000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println("休眠时间到了，执行时间..."+new Date());
    }
}

class MyTask2 implements  Runnable{

    @Override
    public void run(){
        System.out.println("mytask2执行了,执行时间.."+new Date());
    }
}

public class TimerDemo {

    public static void main(String[] args) {



        ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);

        //3秒后开始启动
        scheduledExecutorService.scheduleAtFixedRate(new MyTask1(),3,10, TimeUnit.SECONDS);

        //5秒后开始启动
        scheduledExecutorService.scheduleAtFixedRate(new MyTask2(),5,10,TimeUnit.SECONDS);
    }

}
```

结果：

```java
mytask1执行了,执行时间..Tue Jul 24 00:21:04 CST 2018
mytask2执行了,执行时间..Tue Jul 24 00:21:06 CST 2018
休眠时间到了，执行时间...Tue Jul 24 00:21:09 CST 2018
mytask1执行了,执行时间..Tue Jul 24 00:21:14 CST 2018
mytask2执行了,执行时间..Tue Jul 24 00:21:16 CST 2018
休眠时间到了，执行时间...Tue Jul 24 00:21:19 CST 2018
```

分析结果：我们能看到task2在task1两秒后执行了，而不是等到task1休眠时间到了再去执行。

------

  **2、解决抛出异常时，终止其它定时任务**

```java
class TestJob implements Runnable {

    int count = 0;

    public TestJob() {

    }

    @Override
    public void run() {
        count++;
        System.out.println("TestJob count : " + count);
        if (count == 5) {
            throw new NullPointerException();
        }

    }
}

class TestJob2 implements Runnable {

    int count = 0;

    public TestJob2() {

    }

    @Override
    public void run() {
        count++;
        System.out.println("TestJob2 count : " + count);
    }
}

public class TimerDemo {

    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newSingleThreadScheduledExecutor();
        service.scheduleAtFixedRate(new TestJob(), 2000, 10000, TimeUnit.MILLISECONDS);
        service.scheduleAtFixedRate(new TestJob2(), 3000, 10000, TimeUnit.MILLISECONDS);
    }

}
```

结果：

```java
TestJob count : 1
TestJob2 count : 1
TestJob count : 2
TestJob2 count : 2
TestJob count : 3
TestJob2 count : 3
TestJob count : 4
TestJob2 count : 4
TestJob count : 5
TestJob2 count : 5
TestJob2 count : 6
TestJob2 count : 7
```

从结果分析：TestJob抛出异常结束后，悄然无声的消失，不会影响job2。但是这样不容易排除错误，建议在run函数中加上异常处理。规范写法如下：

```java
class TestJob implements Runnable {

    int count = 0;

    public TestJob() {

    }

    @Override
    public void run() {
        count++;
        System.out.println("TestJob count : " + count);
        try {
            if (count == 5) {
                throw new NullPointerException();
            }
        } catch(Exception e) {
            e.printStackTrace();
        }


    }
}

class TestJob2 implements Runnable {

    int count = 0;

    public TestJob2() {

    }

    @Override
    public void run() {
        count++;
        System.out.println("TestJob2 count : " + count);
    }
}

public class TimerDemo {

    public static void main(String[] args) {
        ScheduledExecutorService service = Executors.newSingleThreadScheduledExecutor();
        service.scheduleAtFixedRate(new TestJob(), 2000, 6000, TimeUnit.MILLISECONDS);
        service.scheduleAtFixedRate(new TestJob2(), 3000, 6000, TimeUnit.MILLISECONDS);
    }

}
```

结果：能看出定时任务出现了异常了。

```java
TestJob count : 1
TestJob2 count : 1
TestJob count : 2
TestJob2 count : 2
TestJob count : 3
TestJob2 count : 3
TestJob count : 4
TestJob2 count : 4
TestJob count : 5
java.lang.NullPointerException
	at com.hj.TestJob.run(TimerDemo.java:26)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.runAndReset(FutureTask.java:308)
	at 
TestJob2 count : 5
TestJob count : 6
TestJob2 count : 6
TestJob count : 7
TestJob2 count : 7
```

## Quartz

虽然ScheduledExecutorService对Timer进行了线程池的改进，但是依然无法满足复杂的定时任务调度场景。因此OpenSymphony提供了强大的开源任务调度框架：Quartz。Quartz是纯Java实现，而且作为Spring的默认调度框架，由于Quartz的强大的调度功能、灵活的使用方式、还具有分布式集群能力，可以说Quartz出马，可以搞定一切定时任务调度！

 **Quartz的体系结构**

![选区_192.png](https://i.loli.net/2018/07/24/5b560645c8806.png)

 

 

 

 