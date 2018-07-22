# lambda表达式

[TOC]

## Lambda表达式的入门

**lambda表达式本质是匿名方法**，下面是一些lambda表达式：

```java
(int x, int y) -> x + y
() -> 42
(String s) -> { System.out.println(s); }
```



第一个lambda表达式接收x和y这两个整形参数并返回它们的和；
第二个lambda表达式不接收参数，返回整数42；
第三个lambda表达式接收一个字符串并把它打印到控制台，不返回值。

## Lambda表达式的结构

- 一个 Lambda 表达式可以有零个或多个参数
- 参数的类型既可以明确声明，也可以根据上下文来推断。例如：`(int a)`与`(a)`效果相同
- 所有参数需包含在圆括号内，参数之间用逗号相隔。例如：`(a, b)` 或 `(int a, int b)` 或 `(String a, int b, float c)`
- 空圆括号代表参数集为空。例如：`() -> 42`
- 当只有一个参数，且其类型可推导时，圆括号（）可省略。例如：`a -> return a*a`
- Lambda 表达式的主体可包含零条或多条语句
- 如果 Lambda 表达式的主体只有一条语句，花括号{}可省略。如果方法需要有返回值连return关键词都可以省略，系统自动将这一行的代码的结果返回。
- 如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空
- 什么时候可以使用Lambda表达式代替匿名内部类，也就是Lambda表达式的应用场景是函数接口

## lambda表达式是怎么知道我们实现的是接口的哪一个方法呢？

`lambda表达式`的类型也被称为`目标类型 target type`，该类型必须是`函数式接口 Functional Interface`，`函数式接口`代表有且只有一个抽象方法，但是可以包含多个默认方法或类方法的接口，因此使用`lambda表达式`系统一定知道我们实现的接口的哪一个方法，因为实现的接口有且只有一个抽象方法供我们实现。

`函数式接口`可以使用注释`@FunctionalInterface`来要求编译器在编译时进行检查，是否只包含一个抽象方法。`Java`提供了大量的`函数式接口`这样就能使用`lambda表达式`简化编程。`lambda表达式`的目标类型必须是`函数式接口`，`lambda表达式`也只能为`函数式接口`创建对象因为`lambda表达式`只能实现一个抽象方法。

 

## lambda的用法

```java
@Test
public void test2(){
    List<String> list = new ArrayList<>();
    list.add("Objective-C");
    list.add("Swift");
    list.add("Python");
    list.add("Golang");
    list.add("Java");
    System.out.println("使用匿名内部类");
    list.sort(new Comparator<String>(){
        @Override
        public int compare(String s1, String s2) {
            return s1.compareTo(s2);
        }
    });
    list.forEach(System.out::println);
    //使用lambda表达式
    System.out.println("使用lambda表达式");
    list.sort((s1,s2)->s1.compareTo(s2));
    list.forEach(System.out::println);

    //使用更简洁的lambda表达式
    System.out.println("使用更简洁的lambda表达式");
    list.sort(String::compareTo); //这个就相当于在compare中调用了compareTo
    list.forEach(System.out::println);
}
```

结果：

```java
使用匿名内部类
Golang
Java
Objective-C
Python
Swift
使用lambda表达式
Golang
Java
Objective-C
Python
Swift
使用更简洁的lambda表达式
Golang
Java
Objective-C
Python
Swift
```

前文介绍了在使用`lambda表达式`时，如果代码体只有一行代码可以省略花括号，如果有返回值也可以省略`return`关键词，不仅如此，`lambda表达式`在只有一条代码时还可以引用其他方法或构造器并自动调用，可以省略参数传递，代码更加简洁，引用方法的语法需要使用`::`符号。`lambda表达式`提供了四种引用方法和构造器的方式：

- 引用对象的方法 `类::实例方法`
- 引用类方法 `类::类方法`
- 引用特定对象的方法 `特定对象::实例方法`
- 引用类的构造器 `类::new`

分析上面例子第三种sort，系统会自动将`函数式接口`实现的方法的所有参数中的第一个参数作为调用者，接下来的参数依次传入引用的方法中即自动进行`s1.compareTo(s2)`的方法调用，明显第三个`sort`函数调用更加简洁明了。`list.forEach(System.out::println);`则引用了类方法，集合类的实例方法`forEach`接收一个`Consumer`接口对象，该接口是一个`函数式接口`，只有一个抽象方法`void accept(T t);`，因此可以使用`lambda表达式`进行调用，这里引用`System.out`的类方法`println`，引用语法`类::类方法`，系统会自动将实现的`函数式接口`方法中的所有参数都传入该类方法并进行自动调用。

## 用lambda表达式实现Runnable

```java
public class LambdaDemo {

    public static void main(String[] args) {
        new Thread(new Runnable() {
            @Override
            public void run() {
                System.out.println("匿名内部类！！");
            }
        }).start();
    }

    @Test
    public void test(){
        new Thread(()-> System.out.println("lambda表达式")).start();
    }
}
```

```
匿名内部类！！
 lambda表达式
```

顺便提一句，通常都会把lambda表达式内部变量的名字起得短一些。这样能使代码更简短，放在同一行。

## 使用lambda表达式对列表进行迭代

```java
@Test
public void test2(){
    List<String> stringList = Arrays.asList("Lambdas", "default method", "stream");

    for (String s :stringList){
        System.out.println(s);
    }

    System.out.println("======java8之后=====");
    stringList.forEach(s -> System.out.println(s));
}
```



```java

```

结果：



## 什么是函数式接口？

所谓函数式接口就是仅仅声明了一个抽象方法的接口。哪怕这个接口有很多默认方法，但是只要该接口只定义了一个抽象方法，那么它就仍然是一个函数式接口

`java.lang.Runnable` 就是一种函数式接口，在 Runnable 接口中只声明了一个方法 `void run()`

对于Java自带的标准库里的大量单一方法接口，很多都已经标记为`@FunctionalInterface`，表明该接口可以作为函数使用。

## Java8常用的几个函数式接口：

 实际上函数式编程分为以下四种接口:

- 断言型函数式接口:`public interface Predicate boolean test(T t)`;
- 消费型函数式接口:`public interface Consumer void accept(T t)`;

- 功能型函数式接口:`public interface Function R apply(T t)`;
- 供给型函数式接口: `public interface Supplier T get()`;

Predicate

```java
package java.util.function;

@FunctionalInterface
public interface Predicate<T> {

    boolean test(T t);
    
    //默认方法，省略
}
```
Consumer

```java
package java.util.function;

@FunctionalInterface

public interface Consumer<T>{

    void accept(T t);

}

```
Function

```java
package java.util.function;
 
@FunctionalInterface
public interface Function<T, R>{
    R apply(T t);
}
```



