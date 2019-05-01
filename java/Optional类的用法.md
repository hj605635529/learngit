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

### 2.2 基础用法
 - `isPresent()`判断值是否不为空，不为空为true

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

- `ifPresent()`  如果option对象保存的值不是null，则调用consumer对象，否则不调用

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

- `orElse()`如果optional对象保存的值不是null，则返回原来的值，否则返回value

```java
public void test2(){
		System.out.println(Optional.ofNullable(123).orElse(456));   //123
		System.out.println(Optional.ofNullable(null).orElse(789));  //456
	}
```

源码：

```java
public T orElse(T other) {
        return value != null ? value : other;
    }
```

- `orElseGet()`如果optional对象保存的值不是null，则返回原来的值， 否则返回一个由指定Supplier接口生成的对象。

```java
@Test
public void test3(){
    List<Integer> list2 = null;
    list2  = Optional.ofNullable(list2).orElseGet(ArrayList::new);
    list2.add(10);
    System.out.println(list2);   //[10]

    //和这效果一样
    List<Integer> list3 = null;
    list3  = Optional.ofNullable(list3).orElse(new ArrayList<Integer>());
    list3.add(10);
    System.out.println(list3);   //[10]
}
```

源码：

```java
public T orElseGet(Supplier<? extends T> other) {
    return value != null ? value : other.get();
}
```

### 2.3 使用map或flatMap从Optional对象中提取和转换值

`map()`如果Optional保存的值不为空，则对Optional中保存的值进行函数运算，返回新的Optional(可以是任何类型)，否则返回一个空的Optional对象。

![image-20190501205503492](https://ws2.sinaimg.cn/large/006tNc79gy1g2m41127whj31cy0ngah5.jpg)

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
        if (!isPresent())  //值为null，返回一个空的Optional对象
            return empty();
        else {
            return Optional.ofNullable(mapper.apply(value));
        }
    }
```

- `flatMap`如果值存在，就对该值执行提供的 mapping 函数调用，返回一个 Optional 类型的值，否则就返 

  回一个空的 Optional 对象 

  ![image-20190501203805133](https://ws2.sinaimg.cn/large/006tNc79gy1g2m3jfp7s6j31gu0pwn5t.jpg)

源码：

```java
//
public<U> Optional<U> flatMap(Function<? super T, Optional<U>> mapper) {
    Objects.requireNonNull(mapper);
    if (!isPresent())
        return empty();
    else {
        return Objects.requireNonNull(mapper.apply(value));
    }
}
```

案例分析：

![image-20190501210915509](https://ws2.sinaimg.cn/large/006tNc79gy1g2m4fsxl20j312j0om78z.jpg)



### 2.4 使用filter剔除特定的值

- `filter()`方法判断Optional对象中保存的值是否满足Predicate，是的话返回原来的Optional对象，不是的话返回空的Optional对象。

![image-20190501212530976](https://ws3.sinaimg.cn/large/006tNc79gy1g2m4wq0xvej31ge08oad8.jpg)

![image-20190501212608221](https://ws1.sinaimg.cn/large/006tNc79gy1g2m4xd0ha2j31gc0fkjyk.jpg)

![image-20190501212806676](https://ws4.sinaimg.cn/large/006tNc79gy1g2m4zf1t1ej31da0lgwl7.jpg)

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

## 3.使用注意

由于Optional类设计时就没特别考虑将其作为类的字段使用，所以它也并未实现 Serializable接口。由于这个原因，如果你的应用使用了某些要求序列化的库或者框架，在 域模型中使用Optional，有可能引发应用程序故障。然而，我们相信，通过前面的介绍，你 已经看到用Optional声明域模型中的某些类型是个不错的主意，尤其是你需要遍历有可能全 部或部分为空，或者可能不存在的对象时。如果你一定要实现序列化的域模型，作为替代方案， 我们建议你像下面这个例子那样，提供一个能访问声明为Optional、变量值可能缺失的接口， 

代码清单如下: 

```java
     public class Person {
         private Car car;
         public Optional<Car> getCarAsOptional() {
             return Optional.ofNullable(car);
         }
    }
```

## 4.经典使用方式

方式一：

```java
//以前写法
public String getCity(User user)  throws Exception{
        if(user!=null){
            if(user.getAddress()!=null){
                Address address = user.getAddress();
                if(address.getCity()!=null){
                    return address.getCity();
                }
            }
        }
        throw new Excpetion("取值错误"); 
    }

//java 8写法(简单的小技巧：如果Function函数返回的是一个Optional类型，一般用flatMap,否则用map)
public String getCity(User user) throws Exception{
    return Optional.ofNullable(user)
                   .map(u-> u.getAddress())
                   .map(a->a.getCity())
                   .orElseThrow(()->new Exception("取指错误"));
}
```

方式二：

```java
//以前的写法
if(user!=null){
    dosomething(user);
}


//java 8写法
 Optional.ofNullable(user)
    .ifPresent(u->{
        dosomething(u);
});
```

方式三：

```java
//以前的写法
public User getUser(User user) throws Exception{
    if(user!=null){
        String name = user.getName();
        if("zhangsan".equals(name)){
            return user;
        }
    }else{
        user = new User();
        user.setName("zhangsan");
        return user;
    }
}

//java 8写法
public User getUser(User user) {
    return Optional.ofNullable(user)
                   .filter(u->"zhangsan".equals(u.getName()))
                   .orElseGet(()-> {
                        User user1 = new User();
                        user1.setName("zhangsan");
                        return user1;
                   });
}

```

