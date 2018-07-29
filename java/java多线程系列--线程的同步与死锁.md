# java多线程系列--线程的同步与死锁

[TOC]

线程的同步与死锁是多线程⾥⾯最需要重点理解的概念。这种操作的核⼼问题在于：每⼀个线程对象轮番强 占资源带来的问题。

## 同步问题的引出

```java
class Mythread6 implements Runnable{

    private int ticket = 10; //一共10张票

    @Override
    public void run(){
        while(this.ticket>0) { // 还有票
            try {
                Thread.sleep(200);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            } // 模拟⽹络延迟
            System.out.println(Thread.currentThread().getName()+",还有" +this.ticket -- +" 张票");
        }
    }
}



public class Thread3Demo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        Mythread6 mythread6 = new Mythread6();
        new Thread(mythread6,"黄牛A").start();
        new Thread(mythread6,"黄牛B").start();
        new Thread(mythread6,"黄牛C").start();
    }

}
```

结果：

```java
黄牛A,还有10 张票
黄牛B,还有9 张票
黄牛C,还有8 张票
黄牛A,还有7 张票
黄牛B,还有6 张票
黄牛C,还有5 张票
黄牛A,还有4 张票
黄牛B,还有3 张票
黄牛C,还有2 张票
黄牛A,还有1 张票
黄牛B,还有0 张票
黄牛C,还有-1 张票
```

这个时候我们发现，票数竟然出现负数，这种问题我们称之为不同步操作。不同步的唯⼀好处是处理速度快（多个线程并发执⾏）

## 同步处理

所谓的同步指的是所有的线程不是⼀起进⼊到⽅法中执⾏，⽽是按照顺序⼀个⼀个进来，如果要想实现这把"锁"的功能，可以采⽤关键字synchronized来处理。使⽤synchronized关键字处理有两种模式：同步代码块、同步⽅法

> - 使⽤同步代码块 : 如果要使⽤同步代码块必须设置⼀个要锁定的对象，所以⼀般可以锁定当前对 象:this

```java
class Mythread6 implements Runnable {

    private int ticket = 100; //一共100张票

    @Override
    public void run() {
        for (int i = 0; i < 100; ++i) {
            //在同一时刻，只允许一个线程进入代码块处理
            synchronized (this) {
                if (this.ticket > 0) { //表示还有票{
                    try {
                        Thread.sleep(200);  //模拟网络延迟
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                    System.out.println(Thread.currentThread().getName() + ",还有" + this.ticket-- + "张票");
                }
            }
        }
    }
}


public class Thread3Demo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        Mythread6 mythread6 = new Mythread6();
        new Thread(mythread6,"黄牛A").start();
        new Thread(mythread6,"黄牛B").start();
        new Thread(mythread6,"黄牛C").start();
    }

}
```

这种⽅式是在⽅法⾥拦截的，也就是说进⼊到⽅法中的线程依然可能会有多个

> * 同步方法

```java
class MyThread implements Runnable {
    private int ticket = 1000; // ⼀共⼗张票
    @Override
    public void run() {
        for (int i = 0; i < 1000; i++) {
            this.sale();
        }
    }
    public synchronized void sale() {
        if (this.ticket > 0) { // 还有票
            try {
                Thread.sleep(20);
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            } // 模拟⽹络延迟
            System.out.println(Thread.currentThread().getName() + ",还有" + this.ti
                    cket-- + " 张票");
        }
    }
}
public class TestDemo {
    public static void main(String[] args) {
        MyThread mt = new MyThread();
        Thread t1 = new Thread(mt, "⻩⽜A");
        Thread t2 = new Thread(mt, "⻩⽜B");
        Thread t3 = new Thread(mt, "⻩⽜C");
        t1.setPriority(Thread.MIN_PRIORITY);
        t2.setPriority(Thread.MAX_PRIORITY);
        t3.setPriority(Thread.MAX_PRIORITY);
        t1.start();
        t2.start();
        t3.start();
    }
}
```

同步虽然可以保证数据的完整性（线程安全操作），但是其执⾏的速度会很慢

观察synchronized锁多对象:

```java
class Sync{
    public synchronized void test() throws InterruptedException {
        System.out.println("test方法开始，当前线程为："+Thread.currentThread().getName());
        Thread.sleep(1000);
        System.out.println("test方法开始结束，当前线程为:"+Thread.currentThread().getName());
    }
}

class MyThread3 extends Thread{

    @Override
    public void run(){
        Sync sync = new Sync();
        try {
            sync.test();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class Thread3Demo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        for (int i = 0; i < 3; ++i){
            Thread thread = new MyThread3();
            thread.start();
        }

    }

}
```

结果：

```java
test方法开始，当前线程为：Thread-0
test方法开始，当前线程为：Thread-2
test方法开始，当前线程为：Thread-1
test方法开始结束，当前线程为:Thread-0
test方法开始结束，当前线程为:Thread-1
test方法开始结束，当前线程为:Thread-2
```

通过上述代码以及运⾏结果我们可以发现，没有看到synchronized起到作⽤，三个线程同时运⾏test()⽅法。 实际上，**synchronized(this)以及⾮static的synchronized⽅法，只能防⽌多个线程同时执⾏同⼀个对象的同步代码段**。即synchronized锁住的是括号⾥的对象，⽽不是代码。对于⾮static的synchronized⽅法，锁 的就是对象本身也就是this。 当synchronized锁住⼀个对象后，别的线程如果也想拿到这个对象的锁，就必须等待这个线程执⾏完成释放 锁，才能再次给对象加锁，这样才达到线程同步的⽬的。即使两个不同的代码段，都要锁同⼀个对象，那么 这两个代码段也不能在多线程环境下同时运⾏。 那么，如果真要锁住这段代码，要怎么做？

- 方式一：锁住同一个对象就OK了

```java
class Sync{
    public synchronized void test() throws InterruptedException {
        System.out.println("test方法开始，当前线程为："+Thread.currentThread().getName());
        Thread.sleep(1000);
        System.out.println("test方法开始结束，当前线程为:"+Thread.currentThread().getName());
    }
}

class MyThread3 extends Thread{
    private Sync sync;
    public MyThread3(Sync sync){
        this.sync = sync;
    }
    @Override
    public void run(){
        try {
            sync.test();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class Thread3Demo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {

        Sync sync = new Sync();

        for (int i = 0; i < 3; ++i){
            Thread thread = new MyThread3(sync);
            thread.start();
        }

    }

}
```

结果：

```java
test方法开始，当前线程为：Thread-0
test方法开始结束，当前线程为:Thread-0
test方法开始，当前线程为：Thread-2
test方法开始结束，当前线程为:Thread-2
test方法开始，当前线程为：Thread-1
test方法开始结束，当前线程为:Thread-1
```



- 每个类只有一个Class对象，因此可以让synchronized锁这个类对应的Class对象

```java
class Sync{
    public  void test() throws InterruptedException {
        synchronized (Sync.class){
            System.out.println("test方法开始，当前线程为："+Thread.currentThread().getName());
            Thread.sleep(1000);
            System.out.println("test方法开始结束，当前线程为:"+Thread.currentThread().getName());
        }

    }
}

class MyThread3 extends Thread{


    @Override
    public void run(){
        try {
            Sync sync = new Sync();
            sync.test();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}

public class Thread3Demo {

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        
        for (int i = 0; i < 3; ++i){
            Thread thread = new MyThread3();
            thread.start();
        }

    }

}
```

结果：

```java
test方法开始，当前线程为：Thread-0
test方法开始结束，当前线程为:Thread-0
test方法开始，当前线程为：Thread-2
test方法开始结束，当前线程为:Thread-2
test方法开始，当前线程为：Thread-1
test方法开始结束，当前线程为:Thread-1
```

上⾯代码⽤synchronized(Sync.class)实现了全局锁的效果。因此，如果要想锁的是代码段，锁住多个对象的 同⼀⽅法，使⽤这种全局锁，锁的是类⽽不是this。 



**static synchronized⽅法，static⽅法可以直接类名加⽅法名调⽤，⽅法中⽆法使⽤this，所以它锁的不是 this，⽽是类的Class对象，所以，static synchronized⽅法也相当于全局锁，相当于锁住了代码段。**

## 同步不具有继承性

如果父类有一个带synchronized关键字的方法，子类继承并重写了这个方法。

但是同步不能继承，所以还是需要在子类方法中添加synchronized关键字。

## synchronized锁重入

“可重入锁”概念是：自己可以再次获取自己的内部锁。比如一个线程获得了某个对象的锁，此时这个对象锁还没有释放，当其再次想要获取这个对象的锁的时候还是可以获取的，如果不可锁重入的话，就会造成死锁。

```java
class Service{
    synchronized public void service1(){
        System.out.println("service1");
        service2();
    }

    synchronized  public void service2(){
        System.out.println("servier2");
        service3();
    }

    synchronized public void service3(){
        System.out.println("service3");
    }
}

class Mythread5 extends Thread{

    @Override
    public void run(){
        Service service = new Service();
        service.service1();
    }
}


public class Thread3Demo {

    public static void main(String[] args) throws InterruptedException {
        new Mythread5().start();
    }

}
```

结果：

```java
service1
servier2
service3
```

可重入锁也支持在父子类继承的环境中：

```java
class Father{

    public int i = 10;

    synchronized public void fatherMethod() throws InterruptedException {
        i--;
        System.out.println("father print i = "+i);
        Thread.sleep(100);
    }

}

class Son extends  Father{
    public int i = 10;

    synchronized public void sonMethod() throws InterruptedException {
       while (i > 0){
           i--;
           System.out.println("son print i = "+i);
           Thread.sleep(100);
           this.fatherMethod();
       }

    }
}

class Mythread5 extends Thread{

    @Override
    public void run(){
        Son son = new Son();
        try {
            son.sonMethod();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}


public class Thread3Demo {

    public static void main(String[] args) throws InterruptedException {
        new Mythread5().start();
    }

}
```

结果：

```java
son print i = 2
father print i = 2
son print i = 1
father print i = 1
son print i = 0
father print i = 0
```

说明当存在父子类继承关系时，子类是完全可以通过“可重入锁”调用父类的同步方法。

另外出现异常时，其锁持有的锁会自动释放。

## 死锁

同步的本质在于：⼀个线程等待另外⼀个线程执⾏完毕才可以继续执⾏。但是如果现在相关的⼏ 个线程彼此之间都在等待着，那么就会造成死锁。

```java
class Pen {
    private String pen = "笔" ;
    public String getPen() {
        return pen;
    }
}
class Book {
    private String book = "本" ;
    public String getBook() {
        return book;
    }
}

public class DeadLock {
    private Pen pen = new Pen();
    private Book book = new Book();

    public void deadLock(){
        Thread thread1 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (pen){
                    System.out.println(Thread.currentThread()+":我有笔，我就不给你");
                    try {
                        Thread.sleep(2000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }

                synchronized (book){
                    System.out.println(Thread.currentThread()+":把你的本子给我!!!");
                }
            }
        },"pen");

        Thread thread2 = new Thread(new Runnable() {
            @Override
            public void run() {
                synchronized (book){
                    System.out.println(Thread.currentThread()+"：我有本子，就不给你");
                    try {
                        Thread.sleep(3000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                synchronized (pen){
                    System.out.println(Thread.currentThread()+":把你的笔给我！！");
                }
            }
        },"book");

        thread1.start();
        thread2.start();
    }

    public static void main(String[] args) {

        new DeadLock().deadLock();

        System.out.println("程序结束。。。。。");
    }
}
```

注：如果你把时间改的越大，你发现死锁的时间越久。

死锁⼀旦出现之后，整个程序就将中断执⾏，所以死锁属于严重性问题。过多的同步会造成死锁，对于资源 的上锁⼀定要注意不要成"环"

## 生产者与消费者模型

⽣产者与消费者是⼀个经典的供求案例，⽽像这种供求案例：provider、consumer。在以后进⾏各种分布式 结构开发之中都会被⼤量的采⽤。

 需求：希望⽣产者负责⽣产数据，⽽⽣产者每⽣产⼀个完整的数据之后，消费者就要把这些数据取⾛。

![选区_227.png](https://i.loli.net/2018/07/29/5b5d9fd750c28.png)

![选区_228.png](https://i.loli.net/2018/07/29/5b5dc14c9ee8b.png)

```java
class Data { // ⽤于数据保存
    private String title ;
    private String note ;
    // flag = true : 表示允许⽣产，但是不允许消费者取⾛
    // flag = fasle : 表示⽣产完毕，允许消费者取⾛，但是不能⽣产
    private boolean flag = true ;
    public synchronized void set(String title, String note) {
        if (this.flag == false) { // 消费者正在消费，等待
            try {
                super.wait();
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        this.title = title ;
        try {
            Thread.sleep(10);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        this.note = note ;
        this.flag = false ; // ⽣产完数据，将flag置为false告诉消费者可以取出数据
        super.notify(); // 唤醒线程
    }
    public synchronized void get() {
        if (this.flag == true) { // ⽣产者正在⽣产，消费者等待
            try {
                super.wait();
            } catch (InterruptedException e) {
                // TODO Auto-generated catch block
                e.printStackTrace();
            }
        }
        try {
            Thread.sleep(20);
        } catch (InterruptedException e) {
            // TODO Auto-generated catch block
            e.printStackTrace();
        }
        System.out.println(this.title + " = " + this.note) ;
        this.flag = true ; // 消费者消费完数据，将flag置为true表示⽣产者可以继续⽣产
        super.notify() ; // 唤醒等待线程

    }
}
```



```java
class DataProvider implements Runnable {
    private Data data ;
    public DataProvider(Data data) {
        super();
        this.data = data;
    }
    @Override
    public void run() {
        for (int i = 0; i < 20 ; i++) {
            if (i % 2 == 0) {
                this.data.set("刘益铭","Java Coder");
            }
            else {
                this.data.set("蛋哥","Linux⼤佬");
            }
        }
    }
}
class DataConsumer implements Runnable {
    private Data data ;
    public DataConsumer(Data data) {
        super();
        this.data = data;
    }
    @Override
    public void run() {
        for (int i = 0; i < 20 ; i++) {
            this.data.get();
        }
    }
}
public class TestDemo {
    public static void main(String[] args) {
        Data data = new Data() ;
        new Thread(new DataProvider(data)).start(); ;
        new Thread(new DataConsumer(data)).start();
    }
}
```

## 数据类型String的常量池属性

在Jvm中具有String常量池缓存的功能

```java
String s1 = "a";
String s2 = "a";
System.out.println(s1==s2); //true
```

上面代码输出为true.**这是为什么呢？**

**字符串常量池中的字符串只存在一份！ 即执行完第一行代码后，常量池中已存在 “a”，那么s2不会在常量池中申请新的空间，而是直接把已存在的字符串内存地址返回给s2。**

因为数据类型String的常量池属性，所以synchronized(string)在使用时某些情况下会出现一些问题，比如两个线程运行 synchronized("abc")｛｝和synchronized("abc")｛｝修饰的方法时，这两个线程就会持有相同的锁，导致某一时刻只有一个线程能运行。所以尽量不要使用synchronized(string)而使用synchronized(object)