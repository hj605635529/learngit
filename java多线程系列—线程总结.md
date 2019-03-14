# java多线程系列—线程总结

[TOC]

## 1.创建线程的4种方式

1）继承Thread类创建线程

2）实现Runnable接口创建线程

3）使用Callable和Future创建线程

4）使用线程池例如用[Executor](https://www.baidu.com/s?wd=Executor&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)框架

```java
//------------------方式一代码---------------------------------
public class MyThread extends Thread{//继承Thread类
    @Override
　　public void run(){

　　//重写run方法
　　}

}

public class Main {

　　public static void main(String[] args){

　　　　new MyThread().start();//创建并启动线程

　　}

}

//----------------------方式二代码----------------------------------
public class MyThread2 implements Runnable {//实现Runnable接口

　　public void run(){

　　//重写run方法

　　}

}

public class Main {

　　public static void main(String[] args){

　　　　//创建并启动线程
　　　   new Thread(new MyThread2()).start();
　　}

}
//---------------------方式三代码----------------------------------
1】创建Callable接口的实现类，并实现call()方法，然后创建该实现类的实例（从java8开始可以直接使用Lambda表达式创建Callable对象）。

2】使用FutureTask类来包装Callable对象，该FutureTask对象封装了Callable对象的call()方法的返回值

3】使用FutureTask对象作为Thread对象的target创建并启动线程（因为FutureTask实现了Runnable接口）

4】调用FutureTask对象的get()方法来获得子线程执行结束后的返回值
--------------------- 
public class CallableTest {
    public static void main(String[] args) {
     
        //因为Callable接口是函数式接口，可以使用Lambda表达式
        FutureTask<Integer> task = new FutureTask<Integer>((Callable<Integer>)()->{
           int i = 0 ;
           for(;i<100;i++){
               System.out.println(Thread.currentThread().getName() + "的循环变量i的值 ：" + i);
           }
           return i;
        });
       for(int i=0;i<100;i++){
           System.out.println(Thread.currentThread().getName()+" 的循环变量i ： + i");
           if(i==20){
               new Thread(task,"有返回值的线程").start();
           }
       }
       try{
           System.out.println("子线程返回值 ： " + task.get());
        }catch (Exception e){
           e.printStackTrace();
        }
    }
}
//--------------------- 方式四------------------------------
略
```

## 2. sleep和wait的区别:

- `sleep`是`Thread`类的方法,`wait`是`Object`类中定义的方法.
- `Thread.sleep`不会导致锁行为的改变, 如果当前线程是拥有锁的, 那么`Thread.sleep`不会让线程释放锁.
- `Thread.sleep`和`Object.wait`都会暂停当前的线程. OS会将执行时间分配给其它线程. 区别是, 调用`wait`后, 需要别的线程执行`notify/notifyAll`才能够重新获得CPU执行时间.

## 3.CAS无锁技术

即比较并替换。 CAS的思想很简单：三个参数，一个当前内存值V、旧的预期值A、即将更新的值B，当且仅当预期值A和内存值V相同时，将内存值修改为B并返回true，否则什么都不做，并返回false。底层都是通过CPU的lock指令锁住一段代码。会出现的问题：

如果变量V初次读取的时候是A，并且在准备赋值的时候检查到它仍然是A，那能说明它的值没有被其他线程修改过了吗？

如果在这段期间曾经被改成B，然后又改回A，那CAS操作就会误认为它从来没有被修改过。针对这种情况，java并发包中提供了一个带有标记的原子引用类`AtomicStampedReference`，它可以通过控制变量值的版本来保证CAS的正确性

https://www.cnblogs.com/java20130722/p/3206742.html

## 4.ThreadLocal

ThreadLocal，很多地方叫做线程本地变量，也有些地方叫做线程本地存储，其实意思差不多。在多线程环境下防止自己的变量被其它线程篡改

Thread 内部维护一个T hreadLocal.ThreadLocalMap threadLocals = null;  ThreadLocalMap中有一个Entry对象，key是ThreadLocal, value是我们传递的值， 然后维护一个Entry[]数组， 

https://www.jianshu.com/p/fb80fddae789

内存泄漏  https://www.jianshu.com/p/1a5d288bdaee

为了安全地使用ThreadLocal，必须要像每次使用完锁就解锁一样，在每次使用完ThreadLocal后都要调用remove()来清理无用的Entry。这在操作在使用线程池时尤为重要。

## 5.ThreadLocal和synchronized的区别

同步机制(synchronized关键字)采用了以“时间换空间”的方式，提供一份变量，让不同的线程排队访问。而ThreadLocal采用了“以空间换时间”的方式，为每一个线程都提供一份变量的副本，从而实现同时访问而互不影响。

# 6生产者消费者的实现

