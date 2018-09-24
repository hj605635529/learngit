# guava之ImmutableList学习

[TOC]

## 1.介绍

不可变集合，顾名思义就是说集合是不可被修改的。集合的数据项是在创建的时候提供，并且在整个生命周期中都不可改变。

**为什么要用immutable对象？ immutable对象有以下的优点:**

> - 1.对不可靠的客户代码库来说，它使用安全，可以在未受信任的类库中安全的使用这些对象 
> - 2.线程安全的：immutable对象在多线程下安全，没有竞态条件 
> - 3.不需要支持可变性, 可以尽量节省空间和时间的开销. 所有的不可变集合实现都比可变集合更加有效的利用内存 (analysis) 
> - 4.可以被使用为一个常量，并且期望在未来也是保持不变的



## 2.类图

![RegularImmutableList.png](https://i.loli.net/2018/09/24/5ba8f2867c4d3.png)

（ **图中的绿色的虚线代表实现，绿色实线代表接口之间的继承，蓝色实线代表类之间的继承。**）

## 3.Maven依赖

```java
<dependency>
    <groupId>com.google.guava</groupId>
    <artifactId>guava</artifactId>
    <version>24.0-jre</version>
</dependency>
```

## 4.Guava集合和不可变集合对应关系

![选区_493.png](https://i.loli.net/2018/09/24/5ba89172db107.png)

## 5.Immutable集合使用用法：

immutable集合都是抽象类可以有以下几种方式来创建：

-  1.用copyOf方法, 譬如, ImmutableSet.copyOf(set)
-  2.使用of方法，譬如，ImmutableList.of("a", "b", "c"),如果元素是null元素，报出空指针异常
- 3.使用Builder类 

## 6.源码剖析

### 1.copyof方法源码分析

```java
  @Test
    public void test1() {
        List<Integer> numbers = Lists.newArrayList(1, 2, 3, 4);

        List<Integer> integers1 = ImmutableList.copyOf(numbers);
        List<Integer> integers2 = Collections.unmodifiableList(numbers);
		ImmutableList<Integer> integers3 = ImmutableList.copyOf(integers1);

		numbers.add(0, -1);

		System.out.println(integers1);
		System.out.println(integers2);
		System.out.println(integers3);

    }
```

结果：

```java
[1, 2, 3, 4]
[-1, 1, 2, 3, 4]
[1, 2, 3, 4]
```

首先让我们来看看最常用的copyOf源码：

```java
public static <E> ImmutableList<E> copyOf(Collection<? extends E> elements) {
  if (elements instanceof ImmutableCollection) {  //集合是不可变集合
    ImmutableList<E> list = ((ImmutableCollection<E>) elements).asList();//将这个不可变集合转换成ImmutableList集合
    return list.isPartialView() ? ImmutableList.<E>asImmutableList(list.toArray()) : list; //将集合中的元素转换为数组
  }
  return construct(elements.toArray());  //如果集合不是Immutable集合，直接调用构造方法，
}
```

所有的immutable集合都以asList()的形式提供了ImmutableList视图（view）。譬如，你把数据放在ImmutableSortedSet，你就可以调用sortedSet.asList().get(k)来取得前k个元素的集合。 返回的ImmutableList常常是个常数复杂度的视图，而不是一个真的拷贝。

```java
private static <E> ImmutableList<E> construct(Object... elements) {
  return asImmutableList(checkElementsNotNull(elements)); //检查每一个元素都不能为空， 如果是null, 报出空指针异常
}
```

```java
static Object[] checkElementsNotNull(Object... array) {
  return checkElementsNotNull(array, array.length);
}

@CanIgnoreReturnValue
static Object[] checkElementsNotNull(Object[] array, int length) {
  for (int i = 0; i < length; i++) {
    checkElementNotNull(array[i], i);
  }
  return array;
}

// We do this instead of Preconditions.checkNotNull to save boxing and array
// creation cost.
@CanIgnoreReturnValue
static Object checkElementNotNull(Object element, int index) {
  if (element == null) {
    throw new NullPointerException("at index " + index);
  }
  return element;
}
```

```java
/**
 * Views the array as an immutable list. Does not check for nulls; does not copy.
 *
 * 数组一定是内部创建的
 */
static <E> ImmutableList<E> asImmutableList(Object[] elements) {
  return asImmutableList(elements, elements.length);
}
```


```java
/**
 *将数组视为不可变列表。 如果指定的范围未覆盖整个数组，则复制。 不检查空值。
 */
static <E> ImmutableList<E> asImmutableList(Object[] elements, int length) {
  switch (length) {
    case 0:
      return of();
    case 1:
      return of((E) elements[0]);
    default:
      if (length < elements.length) {
        elements = Arrays.copyOf(elements, length); //截取数组的一部分
      }
      return new RegularImmutableList<E>(elements);
  }
}
```

```java
class RegularImmutableList<E> extends ImmutableList<E> {
  static final ImmutableList<Object> EMPTY = new RegularImmutableList<>(new Object[0]);

  @VisibleForTesting final transient Object[] array;

  RegularImmutableList(Object[] array) { //这里我们能看出，其实它直接返回原数值的一个引用了
    this.array = array;
  }
```
**总结下copyof方法，能发现其实这个不可变集合内部没有创建内存，都是引用了原来的内存。**

### 2.Builder类构建Immutable集合

```java
@Test
public void test2() {
    ImmutableList.Builder<Integer> builder = new ImmutableList.Builder<Integer>();

    // 绝对不要这样做，初始size为4,后续可能会导致扩容copy多次发生
    ImmutableList<Integer> in = builder.add(1).add(2).add(3).add(4).add(5).add(6).build();
    // 只扩容一次
    ImmutableList<Integer> in2 = builder.add(1, 2, 3, 4, 5, 6).build();

    System.out.println(in);
    System.out.println(in2);
    System.out.println(in);//底层共用一个数组，为啥in2添加元素到数组中，in还是不受影响呢？？因为在asImmutableList(Object[] elements, int length)会调用elements = Arrays.copyOf(elements, length);每次的elements是不同的，
}
```

结果：

```java
[1, 2, 3, 4, 5, 6]
[1, 2, 3, 4, 5, 6, 1, 2, 3, 4, 5, 6]
[1, 2, 3, 4, 5, 6]
```

BuilderUML图：

![Builder.png](https://i.loli.net/2018/09/25/5ba911243415a.png)

- 构造方法：

```java
/**
    *Builder类用于创建不可变列表实例，尤其是{@code public static final}列表
    *（“常数列表”）。 例：
    * public static final ImmutableList <Color> GOOGLE_COLORS
    * = new ImmutableList.Builder <Color>（）
    * .addAll（WEBSAFE_COLORS）
    * .add（new Color（0,191,255））
    * .build（）;
    * 元素以与添加到构建器相同的顺序显示在结果列表中。
    * 可以重用Builder实例; 可以多次调用{@link #build}进行构建
    *多个系列列表。 每个新列表都包含之前创建的所有元素（因为重用builder实例，也就是重用了底层的数组了）
  */

 public static final class Builder<E> extends ImmutableCollection.ArrayBasedBuilder<E> {
   
    public Builder() {
      this(DEFAULT_INITIAL_CAPACITY); //默认容量为4
    }

    Builder(int capacity) {
      super(capacity); //调用父类的构造，父类的构造就是创建一个容量大小为capacity大小的object数组
    }
     
//父类的构造
     abstract static class ArrayBasedBuilder<E> extends ImmutableCollection.Builder<E> {
  Object[] contents;
  int size;
  boolean forceCopy;

  ArrayBasedBuilder(int initialCapacity) {
    checkNonnegative(initialCapacity, "initialCapacity");
    this.contents = new Object[initialCapacity];
    this.size = 0;
  }

    /**
     * 调用build（）方法返回ImmutableList集合
     */
    @Override
    public ImmutableList<E> build() {
      forceCopy = true;
      return asImmutableList(contents, size);
    }
  }
}
```

- add()方法：

```java
 /**
     * Adds {@code element} to the {@code ImmutableList}.
     *
     * @param element the element to add
     * @return this {@code Builder} object
     * @throws NullPointerException if {@code element} is null
     */
    @CanIgnoreReturnValue
    @Override
    public Builder<E> add(E element) {
      super.add(element); //向数组中添加元素， 但是添加元素之前超过初始大小扩容，并且每次按扩大到原来size的1.5倍+1
      return this;
    }

//父类ImmutableCollection.ArrayBasedBuilder<E>的add（）方法
  @CanIgnoreReturnValue
  @Override
  public ArrayBasedBuilder<E> add(E element) {
    checkNotNull(element);
    getReadyToExpandTo(size + 1);
    contents[size++] = element;
    return this;
  }
 private void getReadyToExpandTo(int minCapacity) {
    if (contents.length < minCapacity) { //数组中没有空间放这个新元素了
      this.contents =
          Arrays.copyOf(this.contents, expandedCapacity(contents.length, minCapacity));   //扩容
      forceCopy = false;
    } else if (forceCopy) {
      this.contents = contents.clone();
      forceCopy = false;
    }
  }
//调用父类Builder中的expandedCapacity扩容
 public abstract static class Builder<E> {
    static final int DEFAULT_INITIAL_CAPACITY = 4;

    static int expandedCapacity(int oldCapacity, int minCapacity) {
      if (minCapacity < 0) {
        throw new AssertionError("cannot store more than MAX_VALUE elements");
      }
      // careful of overflow!
      int newCapacity = oldCapacity + (oldCapacity >> 1) + 1;
      if (newCapacity < minCapacity) {
        newCapacity = Integer.highestOneBit(minCapacity - 1) << 1; //扩容后还是空间不够，则最小空间大小就是minCapaacity了。
      }
      if (newCapacity < 0) {
        newCapacity = Integer.MAX_VALUE;
        // guaranteed to be >= newCapacity
      }
      return newCapacity;
    }
```

- add(E... elements)方法

 ```java
  /**
   * Adds each element of {@code elements} to the {@code ImmutableList}.
   *
   * @param elements the {@code Iterable} to add to the {@code ImmutableList}
   * @return this {@code Builder} object
   * @throws NullPointerException if {@code elements} is null or contains a null element
 */
  @CanIgnoreReturnValue
  @Override
  public Builder<E> add(E... elements) {
    super.add(elements);
    return this;
  }

//父类add(...)
  @CanIgnoreReturnValue
    @Override
    public Builder<E> add(E... elements) {
      checkElementsNotNull(elements);
      getReadyToExpandTo(size + elements.length); //最小的空间
      System.arraycopy(elements, 0, contents, size, elements.length);//将所有待插入的元素拷贝到contents数组中
      size += elements.length;
      return this;
    }
 ```
- addAll()方法

```java
   /**
     * Adds each element of {@code elements} to the {@code ImmutableList}.
     *
     * @param elements the {@code Iterable} to add to the {@code ImmutableList}
     * @return this {@code Builder} object
     * @throws NullPointerException if {@code elements} is null or contains a null element
     */
    @CanIgnoreReturnValue
    @Override
    public Builder<E> addAll(Iterable<? extends E> elements) {
      super.addAll(elements);
      return this;
    }


//父类
 public Builder<E> addAll(Iterable<? extends E> elements) {
      if (elements instanceof Collection) {
        Collection<?> collection = (Collection<?>) elements;
        getReadyToExpandTo(size + collection.size());
        if (collection instanceof ImmutableCollection) {
          ImmutableCollection<?> immutableCollection = (ImmutableCollection<?>) collection;
          size = immutableCollection.copyIntoArray(contents, size);
          return this;
        }
      }
      super.addAll(elements);
      return this;
    }

//再调用父类的父类的addAll方法
  public Builder<E> addAll(Iterable<? extends E> elements) {
      for (E element : elements) {
        add(element);   //这里还是一个个添加到数组中， 也会导致多次扩容
      }
      return this;
    }
```

## 7.ImmutableList使用场景

- 初始化集合作为筛选用，黑名单功能
- 防止返回的集合引用，被他人误用，修改原集合，导致bug出现



