# Optional类的用法

[TOC]

## 1.介绍

空指针异常是导致Java应用程序失败的最常见原因。以前，为了解决空指针异常，Google公司著名的Guava项目引入了Optional类，Guava通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。受到Google Guava的启发，Optional类已经成为Java 8类库的一部分。**Optional实际上是个容器：它可以保存类型T的值，或者仅仅保存nul** l。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。

使用Optional而不是null的一个非常重要而又实际的语义区别是，我们 在声明变量时使用的是Optional<Car>类型，而不是Car类型，这句声明非常清楚地表明了这 里发生变量缺失是允许的。与此相反，使用Car这样的类型，可能将变量赋值为null，这意味 着你需要独立面对这些，你只能依赖你对业务模型的理解，判断一个null是否属于该变量的有 效范畴。 

![image-20190501115346270](https://ws2.sinaimg.cn/large/006tNc79gy1g2lodvepk1j316b0u0171.jpg)

## 2.源码剖析

### 2.1 创建Optional对象

-  声明一个空的Optional 

你可以通过静态工厂方法Optional.empty，创建一个空的Optional 对象: 

```java
    Optional<Car> optCar = Optional.empty();
```

-  依据一个非空值创建Optional 你还可以使用静态工厂方法Optional.of，依据一个非空值创建一个Optional对象: 

```java
    Optional<Car> optCar = Optional.of(car);
```

如果car是一个null，这段代码会立即抛出一个NullPointerException，而不是等到你 试图访问car的属性值时才返回一个错误。 

- 可接受null的Optional 

最后，使用静态工厂方法Optional.ofNullable，你可以创建一个允许null值的Optional 对象: 

```java
    Optional<Car> optCar = Optional.ofNullable(car);
```

如果car是null，那么得到的Optional对象就是个空对象。 

- 实例演示

Optional.of()或者Optional.ofNullable()：创建Optional对象，差别在于of不允许参数是null，而ofNullable则无限制

```java
public void test2(){

      Optional<Integer> optional1 = Optional.of(1);
      Integer integer1 = optional1.get();
      System.out.println(integer1);

      Optional<Object> optional2 = Optional.of(null); //这行直接报空指针异常
   }
```

结果：

```java
1

java.lang.NullPointerException
	at java.util.Objects.requireNonNull(Objects.java:203)
```



```java
public void test2(){
   Optional<Integer> optional1 = Optional.ofNullable(2);
   Integer integer = optional1.get();
   System.out.println(integer);

   Optional<Object> o = Optional.ofNullable(null);
   Object o1 = o.get();   //在这行报异常
   System.out.println(o1);
}
```
结果：

```java
2

java.util.NoSuchElementException: No value present
```

源码：

```java
package java.util;

import java.util.function.Consumer;
import java.util.function.Function;
import java.util.function.Predicate;
import java.util.function.Supplier;

/**
 * @since 1.8
 */
public final class Optional<T> {
   
    private static final Optional<?> EMPTY = new Optional<>(); //调用空参构造函数
    private final T value;    //封装的值
    //两个私有的构造函数，只能通过静态方法of或ofNullable去创造这个类型的对象
    private Optional() {
        this.value = null;
    }

     private Optional(T value) {
        this.value = Objects.requireNonNull(value);
    }
    
    public static<T> Optional<T> empty() {      //返回一个空的Optional对象
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
   
    public static <T> Optional<T> of(T value) { //of静态方法， 参数不能为空
        return new Optional<>(value);
    }

    public static <T> Optional<T> ofNullable(T value) {//参数为空，返回一个创建好的empty对象
        return value == null ? empty() : of(value);
    }
    
    public T get() {
        if (value == null) {   //调用空参构造函数的时候，value为null
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
    .......
}
```

### 2.2 使用map从Optional对象中提取和转换值





### isPresent()

判断值是否存在，存在为true

```java
public void test2(){
   Optional<Integer> optional1 = Optional.ofNullable(1);
   Optional<Integer> optional2 = Optional.ofNullable(null);

   System.out.println(optional1.isPresent());
   System.out.println(optional2.isPresent());

}
```
结果：

```java
true
false
```
源码：
```java
//判断值是否存在
    public boolean isPresent() {
        return value != null;
    }
```

### ifPresent()

如果option对象保存的值不是null，则调用consumer对象，否则不调用

```java
public void test2(){

   Optional<Integer> optional1 = Optional.ofNullable(1);
   Optional<Integer> optional2 = Optional.ofNullable(null);

   // 如果不是null,调用Consumer
   optional1.ifPresent(new Consumer<Integer>() {
      @Override
      public void accept(Integer t) {
         System.out.println("value is " + t);
      }
   });

   // null,不调用Consumer
   optional2.ifPresent(new Consumer<Integer>() {
      @Override
      public void accept(Integer t) {
         System.out.println("value is " + t);
      }
   });

}
```
结果：

```java
value is 1
```



源码：

```java
 //如果option对象保存的值不是null，则调用consumer对象，否则不调用
    public void ifPresent(Consumer<? super T> consumer) {
        if (value != null)
            consumer.accept(value);
    }
```

### map()

如果Optional保存的值存在，则对Optional中保存的值进行函数运算，并返回新的Optional(可以是任何类型)

```java
public void test2(){

		Optional<Integer> optional1 = Optional.ofNullable(1);
		Optional<Integer> optional2 = Optional.ofNullable(null);

		Optional<String> str1Optional = optional1.map(new Function<Integer, String>() {
			@Override
			public String apply(Integer a) {
				return "key" + a;
			}
		});
		Optional<String> str2Optional = optional2.map(new Function<Integer, String>() {
			@Override
			public String apply(Integer a) {
				return "key" + a;
			}
		});

		System.out.println(str1Optional.get());// key1
		System.out.println(str2Optional.isPresent());// false

	}
```

结果：

```java
key1
false
```

源码：

```java
  public<U> Optional<U> map(Function<? super T, ? extends U> mapper) {
        Objects.requireNonNull(mapper);
        if (!isPresent())  //值不存在，返回一个空的Optional对象
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

### orElse()

如果optional对象保存的值不是null，则返回原来的值，否则返回value

```java
public void test2(){

		System.out.println(Optional.ofNullable(123).orElse(456));
		System.out.println(Optional.ofNullable(null).orElse(789));

	}
```

结果：

```java
123
789
```

源码：

```java
 public T orElse(T other) {
        return value != null ? value : other;
    }
```

### filter()

判断Optional对象中保存的值是否满足Predicate，是的话返回原来的Optional对象，不是的话返回空的Optional对象。

源码：

```java
 public Optional<T> filter(Predicate<? super T> predicate) {
        Objects.requireNonNull(predicate);
        if (!isPresent())
            return this;
        else
            return predicate.test(value) ? this : empty();
    }
```