# Stream API的使用

[TOC]

## stream API

`Java8`新增的`stream`功能非常强大，这里的`stream`和`Java IO`中的`stream`是完全不同概念的两个东西。本文要讲解的`stream`是能够对集合对象进行各种串行或并发聚集操作，依赖于lambda表达式，只有当两者结合时才能极大的提高编程效率并且代码更易理解和维护。`Stream API`支持串行和并发的集合操作，这也是响应了现在多核处理器的需求，`Stream API`的并发采用的是我们熟悉的`fork/join`模式，手动编写并行代码很复杂也很容易出错，但是采用`Stream API`来进行`集合对象`上的并发操作你不需要编写任何多线程代码就能够轻而易举的实现并发操作，从而提高代码的运行效率，也极大的简化了编程难度。

## 聚集操作

在实际开发中，我们经常对一个集合内的对象进行一系列的操作，比如`排序、查找、过滤、重组、数据统计`等操作，通常情况下我们可能会采用`for循环`遍历的方式来逐一进行操作，这样的代码即复杂又难以维护，如果对性能有要求再进行多线程代码的编写就更加的复杂了，同时也更容易出错。

下面举一个栗子：

```java
class User
{
    private String userID;
    private boolean isVip;
    private int balance;

    public User(String userID, boolean isVip, int balance)
    {
        this.userID = userID;
        this.isVip = isVip;
        this.balance = balance;
    }

    public boolean isVip()
    {
        return this.isVip;
    }

    public String getUserID()
    {
        return this.userID;
    }

    public int getBalance()
    {
        return this.balance;
    }
}

public class HelloWorld
{   
    public static void main(String[] args)
    {
        ArrayList<User> users = new ArrayList<>();
        users.add(new User("2017001", false, 0));
        users.add(new User("2017002", true, 36));
        users.add(new User("2017003", false, 98));
        users.add(new User("2017004", false, 233));
        users.add(new User("2017005", true, 68));
        users.add(new User("2017006", true, 599));
        users.add(new User("2017007", true, 1023));
        users.add(new User("2017008", false, 9));
        users.add(new User("2017009", false, 66));
        users.add(new User("2017010", false, 88));

        //普通实现方式
        ArrayList<User> tempArray = new ArrayList<>();
        ArrayList<String> idArray = new ArrayList<>(3);
        for (User user: users)
        {
            if (user.isVip())
            {
                tempArray.add(user);
            }
        }
        tempArray.sort(new Comparator<User>(){
            public int compare(User o1, User o2) {
                return o2.getBalance() - o1.getBalance();
            }
        });
        for (int i = 0; i < 3; i++)
        {
            idArray.add(tempArray.get(i).getUserID());
        }
        for (int i = 0; i < idArray.size(); i++)
        {
            System.out.println(idArray.get(i));
        }

        //Stream API实现方式
        //也可以使用parallelStream方法获取一个并发的stream，提高计算效率
        Stream<User> stream = users.stream();
        List<String> array = stream.filter(User::isVip).sorted((t1, t2) -> t2.getBalance() - t1.getBalance()).limit(3).map(User::getUserID).collect(Collectors.toList());
        array.forEach(System.out::println);
    }
}
```

上述代码首先定义了一个用户类，这个类保存用户是否是VIP、用户ID以及用户的余额，假如现在有一个需求，将VIP中余额最高的三个用户的ID找出来，传统的思路一般就是创建一个临时的`list`，然后逐一判断，将所有的VIP用户加入到这个临时的`list`中，然后调用集合类的`sort`方法根据余额排序，最后再遍历三次获取余额最高的三个用户的ID等信息。这样的方法看似简单，但代码写出来即混乱也不好看，如果用户量非常大，有几千万甚至几个亿，这样遍历的方式效率就会特别低，如果手工加上多线程的并发操作，代码就更加复杂了。

上述代码的第二部分使用`Stream API`的方式来计算，首先通过集合类获取了一个普通的`stream`，如果数据量大可以使用`parallelStream`方法获取一个并发的`stream`，这样接下来的计算程序员不需要编写任何多线程代码系统会自动进行多线程计算。获取了`stream`以后首先调用`filter`方法找到是否为VIP用户然后对VIP用户进行排序操作，接下来限制只获取三个用户的信息，然后将用户映射为用户ID，最后将该`stream`转换为集合类，两种实现方式的结果完全一样，但是明显的采用`Stream API`的代码更加简洁易懂。

`Stream API`的编写大量依赖`lambda表达式`以及`lambda表达式`的`引用方法`和`引用构造器`

## 如何使用Stream

```
A sequence of elements supporting sequential and parallel aggregate operations1
```

上面是`Java`文档中定义的`Stream`，可以看出，`Stream`就是元素的集合，并且可以采用串行或并行的方式进行聚集操作。在使用时我们可以将`Stream`理解为一个迭代器，只不过这个迭代器更加高级，能够对其中的每一个元素进行我们规定的计算。

当我们要使用`Stream API`时，首先需要创建一个`Stream`对象，可以通过集合类的实例方法`stream`或`parallelStream`来获取一个普通的串行`stream`或是并行`stream`。也可以使用`Stream`、`IntStream`、`LongStream`或`DoubleStream`创建一个`Stream`对象，`Stream`是一个比较通用的流，可以代表任何引用数据类型，其他的则是指特定类型的流。最常用的就是通过一个集合类型来获取相应类型的`Stream`。

流的操作分为`中间操作 Intermediate`和`结束操作 Terminal`：

- 中间操作（Intermediate）：一个流可以采用链式调用的方式进行数个中间操作，主要目的就是打开流然后对这个流进行各种过滤、映射、聚集、统计操作等，如上述代码中的`filter`、`map`操作等。每一个操作结束后都会返回一个新的流，并且这些操作都是lazy的，也就是在进行结束操作时才会真正的进行计算，一次遍历就计算出所有结果。
- 结束操作（Terminal）：一个流只能执行一个结束操作，当执行了结束操作以后这个流就不能再被执行，也就是说不能再次进行中间操作或结束操作，所以结束操作一定是流的最后一个操作，如上述代码中的`collect`方法。当开始执行结束操作的时候才会对流进行遍历并且只一次遍历就计算出所有结果。

## Stream的创建

- 通过集合类创建

通过集合创建`Stream`的方法是我们最常用的，集合类的实例方法`stream`和`parallelStream`可以获取相应的流。

```c++
ArrayList<User> users = new ArrayList<>();
users.add(new User("2017001", false, 0));
users.add(new User("2017002", true, 36));
users.add(new User("2017003", false, 98));
Stream<User> stream = users.stream();  //stream()方法是父类Collection中的方法
```

- 通过数组构造

```java
String[] str = {"Hello World", "Jiaming Chen", "Zhouhang Cheng"};
Stream<String> stream = Stream.of(str);

//方式二
Stream<String> stream2 = Arrays.stream(array);
```

- 通过单个元素构造

```c++
Stream<Integer> stream = Stream.of(1, 2, 3, 4);
```

- Stream与Array和Collection的转换

一般我们都会对`Stream`进行结束操作，用于获取一个数组或是集合类，通过数组和集合类创建`Stream`前文已经介绍了，这里介绍通过`Stream`获取数组或集合类。

```java
String[] str = {"Hello World", "Jiaming Chen", "Zhouhang Cheng"};
Stream<String> stream = Stream.of(str);

String[] strArray = stream.toArray(String[]::new);
List<String> strList = stream.collect(Collectors.toList());　//toList()方法是Ｃollectors的静态方法
ArrayList<String> strArrayList = stream.collect(Collectors.toCollection(ArrayList::new));
Set<String> strSet = stream.collect(Collectors.toSet());
```

上面的代码分别将流转换为数组、List、ArrayList和Set类型，具体的参数可以查看官方API文档。

## Stream 常用方法

- filter

  `filter`的栗子前面已经举过了，`filter`函数需要传入一个实现`Predicate`函数式接口的对象，该接口的抽象方法`test`接收一个参数并返回一个`boolean`值，为`true`则保留，`false`则剔除，前文举的栗子就是判断是否为VIP用户，如果是就保留，不是就剔除。 

  原理如图所示：

  ![选区_184.png](https://i.loli.net/2018/07/22/5b545ce1a2419.png)

- map、flatMap

  `map`方法是一个一对一的映射，每输入一个数据也只会输出一个值。 ![选区_187.png](https://i.loli.net/2018/07/22/5b5461c4035c2.png)

  

   `flatMap`方法是一对多的映射，对每一个元素映射出来的仍旧是一个`Stream`，然后会将这个`子Stream`的元素映射到父集合中，栗子如下： 

```java
public class StreamDemo {


    public static Stream<Character> filterCharacter(String str){

        List<Character> characterList = Lists.newArrayList();

        for (Character ch : str.toCharArray()){
            characterList.add(ch);
        }

        return  characterList.stream();
    }

    @Test
    public void testFlatMap(){

        //map函数生成一个大的stream，把每个小的stream直接放到一个大的stream中。
        Stream<Stream<Character>> streamStream = Arrays.asList("hello", "world", "cccc").stream().map(StreamDemo::filterCharacter);
        streamStream.forEach(sm->{sm.forEach(System.out::println);});

        //flatmap函数生成一个大的strem, 把每个小的stream中的元素放到这个大stream中
        Stream<Character> characterStream = Arrays.asList("hello", "world", "cccc").stream().flatMap(StreamDemo::filterCharacter);
        characterStream.forEach(System.out::println);


    }
}
```

结果：

```java
h
e
l
l
o
w
o
r
l
d

h
e
l
l
o
w
o
r
l
d
```



- limit、skip

`limit`用于限制获取多少个结果，与数据库中的`limit`作用类似，`skip`用于排除前多少个结果。

![选区_188.png](https://i.loli.net/2018/07/22/5b5462498a12b.png)

- sorted

`sorted`的栗子前面也举过了，`sorted`函数需要传入一个实现`Comparator`函数式接口的对象，该接口的抽象方法`compare`接收两个参数并返回一个整型值，作用就是排序，与其他常见排序方法一致。

- distinct

`distinct`用于剔除重复，与数据库中的`distinct`用法一致。

![选区_185.png](https://i.loli.net/2018/07/22/5b5461c3ebc07.png)

- findFirst

`findFirst`方法总是返回第一个元素，如果没有则返回空，它的返回值类型是`Optional<T>`类型，接触过`swift`的同学应该知道，这是一个可选类型，如果有第一个元素则`Optional`类型中保存的有值，如果没有第一个元素则该类型为空。

- min、max

`min`可以对整型流求最小值，返回`OptionalInt`。 
`max`可以对整型流求最大值，返回`OptionalInt`。 

- reduce

`reduce`方法用于组合`Stream`元素，它可以提供一个初始值然后按照传入的计算规则依次和`Stream`中的元素进行计算，因此上文介绍的`min`、`max`都可以看做是`reduce`的一种实现。

reduce方法有三个override的方法：

> Optional<T> reduce(BinaryOperator<T> accumulator);

> T reduce(T identity, BinaryOperator<T> accumulator);

> <U> U reduce(U identity,BiFunction<U, ? super T, U> accumulator,BinaryOperator<U> combiner);

在使用时，我们可以使用Lambada表达式来表示BinaryOperator接口，可以看到reduce方法接受一个函数，这个函数有两个参数，第一个参数是上次函数执行的返回值（也称为中间结果），第二个参数是stream中的元素，这个函数把这两个值相加，得到的和会被赋值给下次执行这个函数的第一个参数。要注意的是：**第一次执行的时候第一个参数的值是Stream的第一个元素，第二个参数是Stream的第二个元素**。这个方法返回值类型是Optional，

```java
public  void test3(){
    Optional accResult = Stream.of(1, 2, 3, 4)
            .reduce((acc, item) -> {
                System.out.println("acc : "  + acc);
                acc += item;
                System.out.println("item: " + item);
                System.out.println("acc+ : "  + acc);
                System.out.println("--------");
                return acc;
            });
    System.out.println("accResult: " + accResult.get());
    System.out.println("--------");
}
```

结果：

```java
acc : 1
item: 2
acc+ : 3
--------
acc : 3
item: 3
acc+ : 6
--------
acc : 6
item: 4
acc+ : 10
--------
accResult: 10
--------
```

下面来看第二个变形，与第一种变形相同的是都会接受一个BinaryOperator函数接口，不同的是其会接受一个identity参数，用来指定Stream循环的初始值。如果Stream为空，就直接返回该值。另一方面，该方法不会返回Optional，因为该方法不会出现null。

```java
int accResult = Stream.of(1, 2, 3, 4)
            .reduce(0, (acc, item) -> {
                System.out.println("acc : "  + acc);
                acc += item;
                System.out.println("item: " + item);
                System.out.println("acc+ : "  + acc);
                System.out.println("--------");
                return acc;
            });
System.out.println("accResult: " + accResult);
System.out.println("--------");
```

结果：

```java
acc : 0
item: 1
acc+ : 1
--------
acc : 1
item: 2
acc+ : 3
--------
acc : 3
item: 3
acc+ : 6
--------
acc : 6
item: 4
acc+ : 10
--------
accResult: 10
```

从打印结果可以看出，reduce前两种变形，因为接受参数不同，其执行的操作也有相应变化：

1. 变形1，未定义初始值，从而第一次执行的时候第一个参数的值是Stream的第一个元素，第二个参数是Stream的第二个元素
2. 变形2，定义了初始值，从而第一次执行的时候第一个参数的值是初始值，第二个参数是Stream的第一个元素

 * count方法

 获取Stream中元素的个数



- Collections.toMap

介绍

```java
 /**
     * Returns a {@code Collector} that accumulates elements into a
     * {@code Map} whose keys and values are the result of applying the provided
     * mapping functions to the input elements.
     *
     * <p>If the mapped keys contains duplicates (according to
     * {@link Object#equals(Object)}), an {@code IllegalStateException} is
     * thrown when the collection operation is performed.  If the mapped keys
     * may have duplicates, use {@link #toMap(Function, Function, BinaryOperator)}
     * instead.
     *
     * @apiNote
     * It is common for either the key or the value to be the input elements.
     * In this case, the utility method
     * {@link java.util.function.Function#identity()} may be helpful.
     * For example, the following produces a {@code Map} mapping
     * students to their grade point average:
     * <pre>{@code
     *     Map<Student, Double> studentToGPA
     *         students.stream().collect(toMap(Functions.identity(),
     *                                         student -> computeGPA(student)));
     * }</pre>
     * And the following produces a {@code Map} mapping a unique identifier to
     * students:
     * <pre>{@code
     *     Map<String, Student> studentIdToStudent
     *         students.stream().collect(toMap(Student::getId,
     *                                         Functions.identity());
     * }</pre> 
* The returned {@code Collector} is not concurrent.  For parallel stream
 * pipelines, the {@code combiner} function operates by merging the keys
 * from one map into another, which can be an expensive operation.  If it is
 * not required that results are inserted into the {@code Map} in encounter
 * order, using {@link #toConcurrentMap(Function, Function)}
 * may offer better parallel performance.
 *
 * @param <T> the type of the input elements
 * @param <K> the output type of the key mapping function
 * @param <U> the output type of the value mapping function
 * @param keyMapper a mapping function to produce keys
 * @param valueMapper a mapping function to produce values
 * @return a {@code Collector} which collects elements into a {@code Map}
 * whose keys and values are the result of applying mapping functions to
 * the input elements
 *
 * @see #toMap(Function, Function, BinaryOperator)
 * @see #toMap(Function, Function, BinaryOperator, Supplier)
 * @see #toConcurrentMap(Function, Function)
 */
public static <T, K, U>
Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                Function<? super T, ? extends U> valueMapper) {
    return toMap(keyMapper, valueMapper, throwingMerger(), HashMap::new);
}






  /**
     * Returns a {@code Collector} that accumulates elements into a
     * {@code Map} whose keys and values are the result of applying the provided
     * mapping functions to the input elements.
     *
     * <p>If the mapped
     * keys contains duplicates (according to {@link Object#equals(Object)}),
     * the value mapping function is applied to each equal element, and the
     * results are merged using the provided merging function.
     *
     * @apiNote
     * There are multiple ways to deal with collisions between multiple elements
     * mapping to the same key.  The other forms of {@code toMap} simply use
     * a merge function that throws unconditionally, but you can easily write
     * more flexible merge policies.  For example, if you have a stream
     * of {@code Person}, and you want to produce a "phone book" mapping name to
     * address, but it is possible that two persons have the same name, you can
     * do as follows to gracefully deals with these collisions, and produce a
     * {@code Map} mapping names to a concatenated list of addresses:
     * <pre>{@code
     *     Map<String, String> phoneBook
     *         people.stream().collect(toMap(Person::getName,
     *                                       Person::getAddress,
     *                                       (s, a) -> s + ", " + a));
     * }</pre>
     *
     * @implNote
     * The returned {@code Collector} is not concurrent.  For parallel stream
     * pipelines, the {@code combiner} function operates by merging the keys
     * from one map into another, which can be an expensive operation.  If it is
     * not required that results are merged into the {@code Map} in encounter
     * order, using {@link #toConcurrentMap(Function, Function, BinaryOperator)}
     * may offer better parallel performance.
     *
     * @param <T> the type of the input elements
     * @param <K> the output type of the key mapping function
     * @param <U> the output type of the value mapping function
     * @param keyMapper a mapping function to produce keys
     * @param valueMapper a mapping function to produce values
     * @param mergeFunction a merge function, used to resolve collisions between
     *                      values associated with the same key, as supplied
     *                      to {@link Map#merge(Object, Object, BiFunction)}
     * @return a {@code Collector} which collects elements into a {@code Map}
     * whose keys are the result of applying a key mapping function to the input
     * elements, and whose values are the result of applying a value mapping
     * function to all input elements equal to the key and combining them
     * using the merge function
     *
     * @see #toMap(Function, Function)
     * @see #toMap(Function, Function, BinaryOperator, Supplier)
     * @see #toConcurrentMap(Function, Function, BinaryOperator)
     */
    public static <T, K, U>
    Collector<T, ?, Map<K,U>> toMap(Function<? super T, ? extends K> keyMapper,
                                    Function<? super T, ? extends U> valueMapper,
                                    BinaryOperator<U> mergeFunction) {
        return toMap(keyMapper, valueMapper, mergeFunction, HashMap::new);
    }



```

```java
//传统java写法

Map<String, Integer> resultMap = new HashMap<>();
for (HotelItem hotelItem : result) {
    if (resultMap.put(hotelItem.hotelSEQ, hotelItem.mobileMinAvailablePrice) != null) {
        throw new IllegalStateException("Duplicate key");
    }
}

//JAVA8 Stream流的应用
 Map<String, Integer> resultMap = result.stream().collect(Collectors.toMap(hotelItem -> hotelItem.hotelSEQ, hotelItem -> hotelItem.mobileMinAvailablePrice));

//总结一下：
//使用Collectors.toMap方法时的两个问题： 
//1、当key重复时，会抛出异常：java.lang.IllegalStateException: Duplicate key 
//2、当value为null时，会抛出异常：java.lang.NullPointerException





---------------------------------------------------------------------------------------

Map<String, Integer> resultMap = Maps.newHashMap();
    for (HotelItem hotelItem : result){
          resultMap.put(hotelItem.hotelSEQ, hotelItem.mobileMinAvailablePrice);
     }


//遇到重复的key就使用后者替换，而且HashMap的value可以是null
  Map<String, Integer> resultMap = result.stream().collect(Collectors.toMap(hotelItem -> hotelItem.hotelSEQ, hotelItem -> hotelItem.mobileMinAvailablePrice, (a, b) -> b));


//lambda表达式
  Map<String, Integer> resultMap = new HashMap<>();
            for (HotelItem hotelItem : result) {
                resultMap.merge(hotelItem.hotelSEQ, hotelItem.mobileMinAvailablePrice, new BinaryOperator<Integer>() {
                    @Override
                    public Integer apply(Integer a, Integer b) {
                        return b;
                    }
                });
            }

    default V merge(K key, V value,
            BiFunction<? super V, ? super V, ? extends V> remappingFunction) {
        Objects.requireNonNull(remappingFunction);
        Objects.requireNonNull(value);
        V oldValue = get(key);
        V newValue = (oldValue == null) ? value :
                   remappingFunction.apply(oldValue, value);
        if(newValue == null) {
            remove(key);
        } else {
            put(key, newValue);
        }
        return newValue;
    }

```



## 案例分析：

![image-20190504173444107](https://ws1.sinaimg.cn/large/006tNc79gy1g2pf3rxwgfj312h0m5adr.jpg)

![image-20190504173529422](https://ws2.sinaimg.cn/large/006tNc79gy1g2pf4is7atj311e0j5tea.jpg)

![image-20190504173610976](https://ws4.sinaimg.cn/large/006tNc79gy1g2pf5316gxj30ye0dgdhc.jpg)