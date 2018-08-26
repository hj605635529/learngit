# Optional类的用法

[TOC]

## 介绍

空指针异常是导致Java应用程序失败的最常见原因。以前，为了解决空指针异常，Google公司著名的Guava项目引入了Optional类，Guava通过使用检查空值的方式来防止代码污染，它鼓励程序员写更干净的代码。受到Google Guava的启发，Optional类已经成为Java 8类库的一部分。**Optional实际上是个容器：它可以保存类型T的值，或者仅仅保存nul** l。Optional提供很多有用的方法，这样我们就不用显式进行空值检测。



## 源码剖析



### Optional.of()或Optional.ofNullable()

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
    
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
   
    public static <T> Optional<T> of(T value) {
        return new Optional<>(value);
    }

    public static <T> Optional<T> ofNullable(T value) {
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