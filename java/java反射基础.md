# java反射基础

[TOC]

**在反射的世界里面,看重的不再是一个对象,而是对象身后的组成(类、构造、普通、成员等)**

## 反射基本语法


### 获得一个类的字节码对象的方式

> 1. 任何类的实例例化对象可以通过Object类中的getClass()方方法取得Class类对象。
> 2. "类.class":直接根据某个具体的类来取得Class类的实例例化对象。
> 3. 使用用Class类提供的方方法:public static Class<?> forName(String className) throws
>   ClassNotFoundException (**最常用**)

```java
package com.hj;

import org.junit.Test;

import java.util.Date;

/**
 * @author jia.huang
 * @create 18-7-15 下午5:37
 */
public class ReflectDemo {

    @Test
    public void test(){
        Date date = new Date();
        Class<? extends Date> dateClass = date.getClass();

        System.out.println(dateClass);

    }

    @Test
    public void test2(){
        Class<?> aClass = null;
        try {
            aClass = Class.forName("java.util.Date");
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        System.out.println(aClass);
    }

    @Test
    public void test3(){
        System.out.println(Date.class);
    }
}
```

结果：

```java
class java.util.Date
class java.util.Date
class java.util.Date
```

### 通过字节码对象获取类的实例

>public T newInstance()   throws InstantiationException, IllegalAccessException

```java
@Test
public void test4(){
    //获取字节码对象
    Class<Date> dateClass = Date.class;
    try {
        //通过字节码对象获取类的实例
        Date date = dateClass.newInstance();
        System.out.println(date);
    } catch (InstantiationException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    }
}
```

## 反射与类操作

### 在字节码对象（Class对象）中一些基本的方法：

- 取得类的包名称: `public Package getPackage()`
- 取得父类的Class对象: `public native Class<? super T> getSuperclass()`
- 取得实现的父接口:`public Class<?>[] getInterfaces()`

```java
@Test
public void test6() throws ClassNotFoundException {
    Class<?> aClass = Class.forName("java.util.Date");

    Package aPackage = aClass.getPackage();
    Class<?> superclass = aClass.getSuperclass();
    Class<?>[] interfaces = aClass.getInterfaces();
    System.out.println(aPackage.getName());
    System.out.println(superclass);

    for (Class<?> cls : interfaces){
        System.out.println(cls);
    }
```

 结果：

```java
java.util
class java.lang.Object
interface java.io.Serializable
interface java.lang.Cloneable
interface java.lang.Comparable
```

### 反射调用构造

* 取得指定参数类型的构造:

> public Constructor<T> getConstructor(Class<?>... parameterTypes)    throws NoSuchMethodException, SecurityException

- 取得类中的所有构造:

>  public Constructor<?>[] getConstructors() throws SecurityException

```java
class Person {
    public Person() {}
    public Person(String name) {}
    public Person(String name,int age) {}
}
class TestPerson {
    public static void main(String[] args) {
        Class<?> cls = Person.class ;
        // 取得类中的全部构造
        Constructor<?>[] constructors = cls.getConstructors() ;
        for (Constructor<?> constructor : constructors) {
            System.out.println(constructor);
        }
    }
}
```

结果：

```java
public com.hj.Person()
public com.hj.Person(java.lang.String)
public com.hj.Person(java.lang.String,int)
```

**==在定义简单java类的时候一定要保留有一个无参构造==**

*Class类通过反射实例化类对象的时候,只能够调用类中的无参构造。如果现在类中没有无参构造则无法使用Class类调用,只能够通过明确的构造调用实例化处理* 

```java
class Person {
   // public Person() {}
    public Person(String name) {}
    public Person(String name,int age) {}

    public void eat(){
        System.out.println("吃吃喝喝");
    }
}
class TestPerson {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException {

        Class<Person> personClass = Person.class;
        Person person = personClass.newInstance();
        person.eat();

    }
}
```

结果：

```java
Exception in thread "main" java.lang.InstantiationException: com.hj.Person
	at java.lang.Class.newInstance(Class.java:427)
	at com.hj.TestPerson.main(ReflectDemo.java:153)
Caused by: java.lang.NoSuchMethodException: com.hj.Person.<init>()
	at java.lang.Class.getConstructor0(Class.java:3082)
	at java.lang.Class.newInstance(Class.java:412)
```

- 得到指定构造方式的构造对象

```java
class TestPerson {
    public static void main(String[] args) throws IllegalAccessException, InstantiationException, InvocationTargetException, NoSuchMethodException {

        Class<Person> personClass = Person.class;
        //得到指定方法的构造器对象
        Constructor<Person> constructor = personClass.getConstructor(String.class, int.class);
        Person huangjia = constructor.newInstance("huangjia", 18);
        huangjia.eat();
    }
}
```

### 反射调用普通方法(核心)

- 取得全部普通方法:

>public Method[] getMethods() throws SecurityException

- 取得指定普通方法:

> public Method getMethod(String name, Class<?>... parameterTypes)

以上两个方法返回的类型是java.lang.reflect.Method类的对象,在此类中提供有一个调用方法的支持:

- 调用

> public Object invoke(Object obj, Object... args)throws IllegalAccessException, Ill
> egalArgumentException,InvocationTargetException

```java
class Person {
    private String name;
    private int age;

    public Person() {
    }

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public String toString() {
        return "Person [name=" + name + ", age=" + age + "]";
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}

class TestPerson {
    public static void main(String[] args) {
        Class<Person> personClass = Person.class;
        Method[] methods = personClass.getMethods();
        for (Method method : methods) {
            System.out.println(method);
        }
    }

}
```

结果:

```java
public int com.hj.Person.getAge()
public void com.hj.Person.setAge(int)
public java.lang.String com.hj.Person.toString()
public java.lang.String com.hj.Person.getName()
public void com.hj.Person.setName(java.lang.String)
public final void java.lang.Object.wait(long,int) throws java.lang.InterruptedException
public final native void java.lang.Object.wait(long) throws java.lang.InterruptedException
public final void java.lang.Object.wait() throws java.lang.InterruptedException
public boolean java.lang.Object.equals(java.lang.Object)
public native int java.lang.Object.hashCode()
public final native java.lang.Class java.lang.Object.getClass()
public final native void java.lang.Object.notify()
public final native void java.lang.Object.notifyAll()
```

- 普通方法的调用

```java
class TestPerson {
    public static void main(String[] args) throws NoSuchMethodException, IllegalAccessException, InstantiationException, InvocationTargetException {
        //得到字节码对象
        Class<Person> personClass = Person.class;
        //任何时候调用类中的普通方法都必须有实例化对象
        Person person = personClass.newInstance();
        // 取得setName这个方法的实例化对象,设置方法名称与参数类型
        Method setName = personClass.getMethod("setName", String.class);
        //相当于Person对象.setName("huangjia") ;
        setName.invoke(person, "huangjia");
        Method getName = personClass.getMethod("getName");
        //// 相当于Person对象.getName() ;
        Object invoke = getName.invoke(person);
        System.out.println(invoke);
    }
}
```

### 反射调用类中的属性

在之前已经成功的实现了类的构造调用、方法调用,除了这两种模式之外还有类中属性调用。
前提:类中的所有属性一定在类对象实例化之后才会进行空间分配,所以此时如果要想调用类的属性,必须保
证有实例化对象。通过反射的newInstance()可以直接取得实例化对象(Object类型)

在Class类中提供有两组取得属性的方法:

>1. 第一组(父类中)-取得类中全部属性: public Field[] getFields() throws SecurityException
>2. 第一组(父类中)-取得类中指定名称属性: public Field getField(String name) throws
>  NoSuchFieldException, SecurityException
>3. 第二组(本类中)-取得类中全部属性: public Field[] getDeclaredFields() throws SecurityException
>4. ​第二组( 本类中)-取得类中指定名称属性 : public Method getDeclaredMethod(String name, Class<?>
>... parameterTypes) throws NoSuchMethodException, SecurityException



```java
 public static void main(String[] args) {
        Class<Person> personClass = Person.class;
        
            // 第一组-取得类中全部属性
            Field[] fields1 = personClass.getFields();
            for (Field field : fields1) {
                System.out.println(field) ;
            }
        System.out.println("------------------------");
        
            // 第二组-取得类中全部属性
            Field[] fields = personClass.getDeclaredFields();
            for (Field field : fields) {
                System.out.println(field);
            }
    }
```

因为在实际开发之中,属性基本上都会进行封装处理,所以没有必要去关注父类中的属性。也就是说以后所取得的属性都以本类属性为主。

而后就需要关注属性的核心描述类:java.lang.reflect.Field,在这个类之中有两个重要方法:

 > 1. 设置属性内容 : public void set(Object obj, Object value) throws IllegalArgumentException,
 >   IllegalAccessException
 > 2. 取得属性内容 : public Object get(Object obj) throws IllegalArgumentException,
 >   IllegalAccessException

- 在AccessibleObject类中提供有一个方法:动态设置封装

  > public void setAccessible(boolean flag) throws SecurityException

- 在Field类之中还有一个特别有用的方法:

  > public Class<?> getType()

```java
class Person {
    private String name;
}

class TestPerson {
    public static void main(String[] args) throws Exception {
        Class<?> cls = Person.class;
        // 实例例化本类对象
        Object obj = cls.newInstance();
        // 操作name属性
        Field nameField = cls.getDeclaredField("name");
        // 抑制Java的访问控制检查
        nameField.setAccessible(true);
        // 相当于对象.name = "huangjia"
        nameField.set(obj, "huangjia");
        // 取得属性
        System.out.println(nameField.get(obj));
        //得到属性的类型
        Class<?> type = nameField.getType();
        System.out.println(type);
    }
} 
```

结果：

```java
huangjia
class java.lang.String
```

![选区_166.png](https://i.loli.net/2018/07/15/5b4b49467b8e6.png)

------

![选区_167.png](https://i.loli.net/2018/07/15/5b4b4ef709023.png)

## ClassLoader类加载器

Class类描述的是整个类的信息,在Class类中提供的forName()方法,这个方法根据ClassPath配置的路径进行类的加载,如果说现在你的类的加载路径可能是网络、文件,这个时候就必须实现类加载器,也就是ClassLoader类的主要作用。

当程序要使用某个类时，如果该类还未被加载到内存中，则系统会通过加载，连接，初始化三步来实现对这个类进行初始化.

### 加载 

1. 就是指将class文件读入内存，并为之创建一个Class对象。

2. 任何类被使用时系统都会建立一个Class对象。

   ![加载.jpg](https://i.loli.net/2018/07/15/5b4b69caaa02c.jpg)


### 链接

   1. 验证 是否有正确的内部结构，并和其他类协调一致
   2. 准备 负责为类的静态成员分配内存，并设置默认初始化值
   3. 解析 将类的二进制数据中的符号引用替换为直接引用

   ### 初始化

略

### 认识ClassLoader

> public ClassLoader getClassLoader()

```java
/**
 * 自定义类,这个类一定在CLASSPATH中
 */

class Member{}

class TestMermber {
    public static void main(String[] args) {
        Class<?> cls = Member.class ;
        System.out.println(cls.getClassLoader()) ;
        System.out.println(cls.getClassLoader().getParent()) ;
        System.out.println(cls.getClassLoader().getParent().getParent());
    }
}
```

   结果：

```java
sun.misc.Launcher$AppClassLoader@18b4aac2
sun.misc.Launcher$ExtClassLoader@5e481248
null
```

此时出现了两个类加载器器:ExtClassLoader(扩展类加载器器)、AppClassLoader(应用程序类加载器) 。

![选区_168.png](https://i.loli.net/2018/07/15/5b4b61f4c3169.png)

Bootstrap(引擎加载器、核心加载器):加载系统核心类
ExtClassLoader(CLASSPATH):加载第三方程序类库。对于第三方程序类库(Ext加载器器)除了CLASSPATH之
外,在Java里面还有一个加载录:/Library/Java/JavaVirtualMachines/jdk1.8.0_20.jdk/Contents/Home/jre/lib/ext。
AppClassLoader(自己写的代码):程序加载器。

以上三种类加载器加载的代码都必须要求在CLASSPATH中加载。

自定义类加载器:用户决定类从哪里加载

ClassLoader类中提供有如下方法(进行类的加载):

> protected Class<?> loadClass(String name, boolean resolve)
> throws ClassNotFoundException

```java
class Member{
    @Override
    public String toString() {
        return "Member";
    }
}
class TestMember{
    public static void main(String[] args) throws Exception{
        Class<?> member = Class.forName("com.hj.Member");
        //得到类加载器
        ClassLoader classLoader = member.getClassLoader();
        //进行类的加载
        Class<?> aClass = classLoader.loadClass("com.hj.Member");
        Object newInstance = aClass.newInstance();
        System.out.println(newInstance.toString());
    }
}
```

结果：

```java
Member
```



## 反射与Annotation

Annotation是颠覆性的开发技术,在以后的开发之中我们会见到大量的Annotation。Annotation的设计有一个前提:需要有代码容器,才可以实现自定义的Annotation。

### 反射取得Annotation信息

Annotation注解可以定义在类或方法上,在学习反射的概念后,此时我们可以通过反射取得所定义的Annotation信息。在java.lang.reflect.AccessibleObject(java.lang.Class)类中提供有如下与Annotation有关的方法:

>1. 取得全部Annotation: public Annotation[] getAnnotations()
>2. 取得指定的Annotation: public T getAnnotation(Class annotationClass)

取得定义在类上的Annotation:

```java
@SuppressWarnings("serial")
@Deprecated
class Animal  {
}

 class TestAnnotation {
    public static void main(String[] args) throws NoSuchMethodException {
        Annotation[] annotations = Animal.class.getAnnotations();

        for (Annotation annotation : annotations){
            System.out.println(annotation);
        }
    }
}
```

Annotation本身有自己的保存范围,不同的Annotation的返回也不同。所以此处只出现了一个Annotation。

在方法上使用Annotation：

```java
@SuppressWarnings("serial")
@Deprecated
class Animal  {
    @Deprecated
    @Override
    public String toString() {
        return super.toString();
    }
}

 class TestAnnotation {
    public static void main(String[] args) throws NoSuchMethodException {
        Annotation[] annotations = Animal.class.getMethod("toString").getAnnotations();

        for (Annotation annotation : annotations){
            System.out.println(annotation);
        }
    }
}
```

自定义Annotation:

要想自定义Annotation,首先需要解决的就是Annotation的作用范围。通过第一节的范例我们可以发现,不同的Annotation有自己的运行范围,而这些范围就在一个枚举类(java.lang.annotation.RetentionPolicy)中定义:

>1. SOURCE:在源代码中出现的Annotation
>2. CLASS:在*.class中出现的Annotation
>3. RUNTIME:在类执行的时候出现的Annotation

取得某个具体的Annotation:

```java
/**
 * 自定义一一个Annotation
 */
@Retention(RetentionPolicy.RUNTIME)
@interface MyAnnotation{
    public String name()  default "huangjia";
    public int age()  default 22;
}
@Deprecated
@MyAnnotation
class Member2 implements Serializable {
}

 class TestAnnotation2 {
    public static void main(String[] args) {
        MyAnnotation annotation = Member2.class.getAnnotation(MyAnnotation.class);
        String name = annotation.name();
        int age = annotation.age();
        System.out.println("name="+name+":"+"age="+age);
    }
}
```

结果：

```java
name=huangjia:age=22
```

## 反射与泛型

拿一个例子来说，比如定义一个字符串类型的List，`List<String> list`，如果我们想往里面添加一个`Integer`类型的元素，能添加进去么？正常情况下，是不行的，因为编译的时候就直接提示错误了，不过通过反射我们可以实现：

```java
@Test
public void test7(){
    List<String> stringList = new ArrayList<String>();

    stringList.add("hello");
  //  stringList.add(10);

    Class<? extends List> aClass = stringList.getClass();
    try {
        Method add = aClass.getMethod("add", Object.class);
        add.invoke(stringList,10);
    } catch (NoSuchMethodException e) {
        e.printStackTrace();
    } catch (IllegalAccessException e) {
        e.printStackTrace();
    } catch (InvocationTargetException e) {
        e.printStackTrace();
    }
    System.out.println(stringList);
}
```

结果：

```java
[hello, 10]
```

**因为泛型是用于编译期间的，编译过后泛型擦除，所以我们才可以借助反射来实现**

## JDK 7处理反射方法的异常

在JDK 7之前，当调用一个反射方法时，不得不捕获多个不相关的检查期异常，比如上面那个例子。JDK 7引入了一个新的反射操作相关异常的父类 `ReflectiveOperationException`，这样的话就可以通过这一异常来捕获所有其他反射操作相关的子类异常，从而使我们的代码更加简洁：

```java
public void test7(){
    List<String> stringList = new ArrayList<String>();

    stringList.add("hello");
  //  stringList.add(10);

    Class<? extends List> aClass = stringList.getClass();
    try {
        Method add = aClass.getMethod("add", Object.class);
        add.invoke(stringList,10);
    } catch (ReflectiveOperationException e) { 
        e.printStackTrace();
    }
    System.out.println(stringList);
}
```

## 总结

  到这里，有关反射的内容基本就学习完了，现在来简单总结一下。反射增加了程序的灵活性，所以在一般的框架中使用比较多，如Spring等。反射的功能很强大，在一定程度上可以说是破坏了Java语言封装的特性，另外反射调用的时候可以忽略权限的检查，从而可能会导致对象的安全性问题。不过，我们不妨换一个角度来思考，思考下什么是封装？什么是安全？

所谓封装，就是将具体的实现细节隐藏，将实现后的结果通过共有方法返回给外部调用，而针对私有方法，即使别人能通过反射的方式调用，但即使调用但却得不到一个完整的结果，因为一般情况下只有共有方法的返回才是完整的。从这一点来说，封装性其实没有被破坏。

而所谓安全，如果是为了保护源码的话，那其实没必要，即使不通过反射，也有其他方式获取源码。

因为Java语言毕竟是一种静态语言，为了让语言拥有动态的特性，必须要有反射机制，而反射机制本身就是底层的处理，不可能按常规的封装特性来处理。也就是说不给调用私有方法的能力，很多程序受到局限，那么实现起来就麻烦了。

所以说我们可以认为，反射机制只是提供了一种强大的功能，使得开发者能在封装之外，按照特定的需要实现一些功能。没有太多的必要纠结于反射是否安全等问题。

 

 

 

 

 

 

 

 

 

 












