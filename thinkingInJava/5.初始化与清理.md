# # 5.初始化与清理

[TOC]

## 重载

- 为啥要方法重载？函数返回值为啥不能作为重载的条件？

  拿构造函数来说吧， 我们可能需要不同的方式来构造一个对象，但是构造函数名字和类名相同，如果不是方法重载， 更本实现不了不同的方式构造对象

  > void f()
  >
  > int f()
  >
  > 假设函数被调用写成  f()  你知道调用那个函数吗

- 重载涉及到的基本类型

  基本类型能从一个“较小”的类型自动提升到一个“较大”的类型，提升规则如下：

  int   -----> long ---->float ------> double

  char ----->int------->long------>float------>double  (char类型略有不同，char直接提升到int,不会提升到short)

  byte------>short------>int------>long------>float-------->double

  short----->int------->long------>float----->double

  long----->float------>double

## this关键字

- ```java
  public class Demo1 {
  
   void f(float a){
      System.out.println("float "+a);
   }
   @Test
   public  void test1(){
      Demo1 d1 = new Demo1();
      Demo1 d2 = new Demo1();
      d1.f(10);
      d2.f(20);
   }
  }
  ```
>f（float a)函数怎么知道它是被那个对象掉用呢？ 其实在内部的实现如：

>Demo1.f(d1,10);
>
>Demo1.f(d2,20)
>
>将对象的引用作为第一个参数传递给方法
- 可以在其他构造函数中通过this()调用对应的构造函数，不能调用两个，并且只能放在第一行，要不然编译器会报错

## 清理

- 垃圾回收器只知道释放那些由new分配的内存

- finalize()方法，有点类似C++的析勾函数  工作原理：一旦垃圾回收器准备释放对象占用的存储空间，将首先调用finaize()方法，并在下次垃圾回收动作发生时，才会真正回收对象占用的内存

- **垃圾回收器工作原理**：

  java虚拟机会进行监视，如果所有对象都很稳定，垃圾回收器的效率降低的话，就切换到**标记-清扫**方式，同样，java虚拟机会跟踪**标记-清扫**的效果，要是堆空间出现很多碎片，就会切换回"**停止-复制**"方式，这就是**自适应技术**

  **标记-清扫模式**：从堆栈和静态存储区出发，遍历所有引用，进而找出所有存活的对象， 每当它找到一个存活对象,就给对象设计一个标记，这个过程不会回收任何对象， 只有全部标记工作完成的时候， 清理动作才会开始。在清理过程中，没有标记的对象将被释放，所以剩下的堆空间是不连续的。

  **停止-复制方式:** 先暂停程序的运行（所以它不属于后台回收模式，标记-清扫工作也需要程序暂停的情况下才能进行），把存活的对象从旧堆复制到新堆，但是会导致大量内存复制行为。因此java虚拟机，内存分配以较大的"块"为单位，对象较大，占用单独的块，有了块之后，垃圾回收器在回收的时候就可以往废弃的块里拷贝对象了，什么是废弃的块？每个块都用相应的代数来记录它是否还存活。块被引用，代数加1.如何减少复制呢? 大型对象不会被复制，只是代数会加1, 内含小型对象的那些块被复制并整理。

## 初始化

- 对于方法的局部变量，在使用之前必须初始化或赋值，要不然报编译错误。

  ```java
  @Test
  public  void test1(){
     int i ;
     System.out.println(i);  //Error:(19, 36) java: 可能尚未初始化变量i
  }
  
  @Test
  public  void test2(){
     int i ;
     i = 20;
     System.out.println(i);   //i = 20
  }
  
  ```

- 在类的内部，变量定义的顺序决定初始化的顺序，即使变量的定义散布方法定义之间，它们仍旧会在任何方法（包括构造器)被调用之前得到初始化。

- static 关键字不能应用于局部变量，他只能作用于域，如果一个域是静态的基本类型，默认初始化是基本类型的标准初值，如果是一个对象引用，默认初始化值是null。

  ```java
  @Test
  public  void test1(){
     static int i = 10;  //Error:(18, 17) java: 非法的表达式开始
     System.out.println(i);
  }
  ```

- 总结下对象的创建过程，假设有个名字为Dog的类：

  - 即使没有显示的使用static关键字，构造器实际上也是一个静态方法，当首次创建类型为Dog类型的对象时（构造器可以看成静态方法），或者Dog类的静态方法/静态域首次被访问时，java解释器必须查找类路径，以定位Dog.class文件。
  - 然后载入Dog.class文件，静态初始化执行，静态初始化只在Class对象首次被加载的时候执行一次。
  - 当用new Dog()创建对象的时候，在堆上分配存储空间。
  - 空间清0, 导致基本类型设置成默认值， 引用类型设置成null
  - 执行字段定义处的初始化动作
  - 执行构造器

```java
class Test1 {
    Test1() {  System.out.println("test1构造函数");}
}
class Test2 { 
    Test2() { System.out.println("test2构造函数");}
}

public class Demo1 {
    public static void main(String[] args) {
      System.out.println("main 被执行了");    //只是执行了前2步
    }
    private Test1 test1 = new Test1();
    static private Test2 test2 = new Test2();
}

//test2构造函数
//main 被执行了
```

## 可变参数

```java
public class Demo1 {

   static void f(float i, Character... args){
      System.out.println("first");
   }

   static void f(Character... args){//改成static void f( char c, Character... args) ok
      System.out.println("second");
   }

   public static void main(String[] args) {
      f('a', 'b');   //Error:(32, 17) java: 对f的引用不明确
                     // com.init.Demo1 中的方法 f(float,java.lang.Character...) 和                          //com.init.Demo1 中的方法 f(java.lang.Character...) 都匹配
   }
}
```

**你应该总是在重载方法的一个版本上使用可变参数列表， 或者压根不使用它**

