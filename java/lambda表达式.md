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
- 如果 Lambda 表达式的主体只有一条语句，花括号{}可省略。匿名函数的返回类型与该主体表达式一致
- 如果 Lambda 表达式的主体包含一条以上语句，则表达式必须包含在花括号{}中（形成代码块）。匿名函数的返回类型与代码块的返回类型一致，若没有返回则为空
- 什么时候可以使用Lambda表达式代替匿名内部类，也就是Lambda表达式的应用场景是函数接口

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

## 什么是函数式接口？

所谓函数式接口就是仅仅声明了一个抽象方法的接口。哪怕这个接口有很多默认方法，但是只要该接口只定义了一个抽象方法，那么它就仍然是一个函数式接口

`java.lang.Runnable` 就是一种函数式接口，在 Runnable 接口中只声明了一个方法 `void run()`

对于Java自带的标准库里的大量单一方法接口，很多都已经标记为`@FunctionalInterface`，表明该接口可以作为函数使用。

## Java8常用的几个函数式接口：

 

- Predicate

  ```java
  package java.util.function;
  
  @FunctionalInterface
  public interface Predicate<T> {
  
      boolean test(T t);
      
      //默认方法，省略
  }
  ```

- Consumer

  ```java
  package java.util.function;
  
  @FunctionalInterface
  
  public interface Consumer<T>{
  
      void accept(T t);
  
  }
  
  ```

- Function

  ```java
  package java.util.function;
   
  @FunctionalInterface
  public interface Function<T, R>{
      R apply(T t);
  }
  ```

  