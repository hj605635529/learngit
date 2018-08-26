# ArrayList源码剖析（1.8）

[TOC]

## 1 ArrayList的概述

### 1.1 ArrayList的基本特点

1.ArrayList底层是一个动态扩容的数组结构

2.允许存放（不止一个）null元素

3.允许存放重复数据，存储顺序按照元素的添加顺序

4.ArrayList并不是一个线程安全的集合。如果集合的增删操作需要保证线程的安全性，可以考虑使用CopyOnWriteArrayList或者使用collections.synchronizedList(List l)函数返回一个线程安全的ArrayList类

###  1.2 ArrayList的继承关系

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
```

从 `ArrayList` 的继承关系来看， `ArrayList` 继承自 `AbstractList`，实现了`List<E>, RandomAccess, Cloneable, java.io.Serializable` 接口。

- List接口定义了列表必须实现的方法。 AbstractList提供了List接口的默认实现（个别方法为抽象方法）。
- RandomAccess是一个标记接口，接口内没有定义任何内容。针对 ArrayList 而言通过 `get(index)`去访问元素可以达到 O(1) 的时间复杂度。有些集合类不拥有这种随机快速访问的能力，比如 `LinkedList` 就没有实现这个接口。
- `ArrayList` 实现 `Cloneable` 接口标识着他可以被克隆/复制，其内部实现了 clone 方法供使用者调用来对 ArrayList 进行克隆，但其实现只通过 `Arrays.copyOf` 完成了对 ArrayList 进行「浅复制」，也就是你改变 `ArrayList clone`后的集合中的元素，源集合中的元素也会改变。
- 对于 `java.io.Serializable` 标识着集合可被被序列化。

我们发现了一些有趣的事情，除了`List<E>` 以外，`ArrayList` 实现的接口都是标识接口，标识着这个类具有怎样的特点，看起来更像是一个属性。

## 2 ArrayList的构造函数

在说构造方法之前我们要先看下与构造参数有关的几个全局变量：

```java
/**
 * ArrayList 默认的数组容量
 */
 private static final int DEFAULT_CAPACITY = 10;

/**
 * 这是一个共享的空的数组实例，当使用 ArrayList(0) 或者 ArrayList(Collection<? extends E> c) 
 * 并且 c.size() = 0 的时候讲 elementData 数组讲指向这个实例对象。
 */
 private static final Object[] EMPTY_ELEMENTDATA = {};

/**
 * 另一个共享空数组实例，再第一次 add 元素的时候将使用它来判断数组大小是否设置为 DEFAULT_CAPACITY
 */
 private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
 * 真正装载集合元素的底层数组 
 *transient是Java语言的关键字，用来表示一个域不是该对象串行化的一部分。当一个对象被串行化的时候，transient型变量的值不包括在串行化的表示中，然而非transient型的变量是被包括进去的。
 */
transient Object[] elementData; // non-private to simplify nested class access
```

看个有关transient的例子：

```java
package com.hj.List;

import java.io.*;

/**
 * @author jia.huang
 * @create 18-8-26 下午5:24
 */

class UserInfo implements Serializable {
    private static final long serialVersionUID = 996890129747019948L;
    private String name;
    private transient String psw;

    public UserInfo(String name, String psw) {
        this.name = name;
        this.psw = psw;
    }

    @Override
    public String toString() {
        return "name=" + name + ", psw=" + psw;
    }
}

public class TestTransient {

    public static void main(String[] args) {
        UserInfo userInfo = new UserInfo("张三", "123456");
        System.out.println(userInfo);
        try {
            // 序列化，被设置为transient的属性没有被序列化
            ObjectOutputStream o = new ObjectOutputStream(new FileOutputStream("UserInfo.out"));
            o.writeObject(userInfo);
            o.close();
        } catch (Exception e) {
            // TODO: handle exception
            e.printStackTrace();
        }
        try { // 重新读取内容
            ObjectInputStream in = new ObjectInputStream(new FileInputStream("UserInfo.out"));
            UserInfo readUserInfo = (UserInfo) in.readObject();
            // 读取后psw的内容为null
            System.out.println(readUserInfo.toString());
        } catch (Exception e) {
            // TODO: handle exception
            e.printStackTrace();
        }
    }
}
```

结果：

```java
name=张三, psw=123456
name=张三, psw=null
```



ArrayList 一共三种构造方式，我们先从无参的构造方法来开始：

### 2.1 无参构造方法

```java
/**
 * 构造一个初始容量为10的空列表。
 */
public ArrayList() {
   this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
```

这是我们经常使用的一个构造方法，其内部实现只是将 `elementData` 指向了我们刚才讲得 `DEFAULTCAPACITY_EMPTY_ELEMENTDATA` 这个空数组，这个空数组的容量是 0， 但是源码注释却说这是构造一个初始容量为10的空列表。这是为什么？其实在集合调用 add 方法添加元素的时候将会调用 `ensureCapacityInternal` 方法，在这个方法内部判断了：

```java
if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
       minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
}
```

可见，如果采用无参数构造方法的时候第一次添加元素肯定走进 if 判断中 minCapacity 将被赋值为 10，所以「构造一个初始容量为10的空列表。」也就是这个意思。

###  2.2 指定初始容量的构造方法

```java
/**
 * 构造一个具有指定初始容量的空列表。
 * @param  初始容量 
 * @throws 如果参数小于 0 将会抛出 IllegalArgumentException  参数不合法异常
 */
public ArrayList(int initialCapacity) {
   if (initialCapacity > 0) {
       this.elementData = new Object[initialCapacity];
   } else if (initialCapacity == 0) {
       this.elementData = EMPTY_ELEMENTDATA;
   } else {
       throw new IllegalArgumentException("Illegal Capacity: "+
                                          initialCapacity);
   }
}
```

如果我们预先知道一个集合元素的容纳的个数的时候推荐使用这个构造方法，比如我们有个ArrayList集合一共需要装 15 个 元素 ，那么我们就可以在构造集合的时候生成一个初始容量为 15 的一个集合。有人会认为 `ArrayList` 自身具有动态扩容的机制，无需这么麻烦，下面我们讲解扩容机制的时候我们就会发现，每次扩容是需要有一定的内存开销的，而这个开销在预先知道容量的时候是可以避免的。

源代码中指定初始容量的构造方法实现，判断了如果 我们指定容量大于 0 ，将会直接 new 一个数组，赋值给 `elementData` 引用作为集合真正的存储数组，而指定容量等于 0 的时候将使用成员变量 `EMPTY_ELEMENTDATA` 作为暂时的存储数组，这是 `EMPTY_ELEMENTDATA` 这个空数组的一个用处（不必太过于纠结 EMPTY_ELEMENTDATA 的作用，其实它的在源码中出现的频率并不高）。

### 2.3 使用另一个集合 Collection 的构造方法

```java
/**
 * 构造一个包含指定集合元素的列表，元素的顺序由集合的迭代器返回。
 *
 * @param 源集合，其元素将被放置到这个集合中。 
 * @如果参数为 null，将会抛出 NullPointerException 空指针异常
 */
 public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray 可能(错误地)不返回 Object[]类型的数组 参见 jdk 的 bug 列表(6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // 如果集合大小为空将赋值为 EMPTY_ELEMENTDATA 等同于 new ArrayList(0);
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

## 3 ArrayList的添加元素&扩容机制

### 3.1 在集合末尾添加一个元素的方法

```java
//成员变量 size 标识集合当前元素个数初始为 0
int size；
/**
 * 将指定元素添加到集合（底层数组）末尾
 * @param 将要添加的元素
 * @return 返回 true 表示添加成功
 */
 public boolean add(E e) {
    //检查当前底层数组容量是否还有位置存放新的元素
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //将数组添加一个元素，size 加 1
    elementData[size++] = e;
    return true;
 }
```

 调用 add 方法的时候总会调用 `ensureCapacityInternal` 来判断是否需要进行数组扩容，`ensureCapacityInternal` 参数为当前集合长度 `size + 1`，这很好理解，是否需要扩充长度，需要看当前底层数组是否够放 `size + 1` 个元素的

### 3.2 扩容机制

```java
//扩容检查
private void ensureCapacityInternal(int minCapacity) {
    //如果是无参构造方法构造的的集合，第一次添加元素的时候会满足这个条件 minCapacity 将会被赋值为 10 如果是通过带有0容量的构造方法构造的集合，minCapacity就会赋值为1,则就是DEFAULTCAPACITY_EMPTY_ELEMENTDATA出现的原因。
   if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
       minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
   }
    // 将 size + 1 或 10 传入 ensureExplicitCapacity 进行扩容判断
   ensureExplicitCapacity(minCapacity);
}

private void ensureExplicitCapacity(int minCapacity) {
  //操作数加 1 用于保证并发访问 
   modCount++;
   // 如果 当前数组的长度比添加元素后的长度要小则进行扩容 
   if (minCapacity - elementData.length > 0)
       grow(minCapacity);
}
```

上边的源码主要做了扩容前的判断操作，注意参数为当前集合元素个数+1，第一次添加元素的时候 `size + 1 = 1` ,而 `elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA`, 长度为 0 ，`1 - 0 > 0`, 所以需要进行 grow 操作也就是扩容。

```java
/**
 * 集合的最大长度 Integer.MAX_VALUE - 8 是为了减少出错的几率 Integer 最大值已经很大了
 */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
 * 增加容量，以确保它至少能容纳最小容量参数指定的元素个数。
 * @param 满足条件的最小容量
 */
private void grow(int minCapacity) {
  //获取当前 elementData 的大小，也就是 List 中当前的容量
   int oldCapacity = elementData.length;
   //oldCapacity >> 1 等价于 oldCapacity / 2  所以新容量为当前容量的 1.5 倍
   int newCapacity = oldCapacity + (oldCapacity >> 1);
   //如果扩大1.5倍后仍旧比 minCapacity 小那么直接等于 minCapacity
   if (newCapacity - minCapacity < 0)
       newCapacity = minCapacity;
    //如果新数组大小比  MAX_ARRAY_SIZE 就需要进一步比较 minCapacity 和 MAX_ARRAY_SIZE 的大小
   if (newCapacity - MAX_ARRAY_SIZE > 0)
       newCapacity = hugeCapacity(minCapacity);
   // minCapacity通常接近 size 大小
   //使用 Arrays.copyOf 构建一个长度为 newCapacity 新数组 并将 elementData 指向新数组
   elementData = Arrays.copyOf(elementData, newCapacity);
}

/**
 * 比较 minCapacity 与 Integer.MAX_VALUE - 8 的大小如果大则放弃-8的设定，设置为 Integer.MAX_VALUE 
 */
private static int hugeCapacity(int minCapacity) {
   if (minCapacity < 0) // overflow
       throw new OutOfMemoryError();
   return (minCapacity > MAX_ARRAY_SIZE) ?
       Integer.MAX_VALUE :
       MAX_ARRAY_SIZE;
}
```

由此看来 ArrayList 的扩容机制的知识点一共又两个

1. **每次扩容的大小为原来大小的 1.5倍 （当然这里没有包含 1.5倍后大于 MAX_ARRAY_SIZE 的情况）**
2. **扩容的过程其实是一个将原来元素拷贝到一个扩容后数组大小的长度新数组中。所以 ArrayList 的扩容其实是相对来说比较消耗性能的。**

### 3.3 在指定角标位置添加元素的方法

```java
/**
* 将指定的元素插入该列表中的指定位置。将当前位置的元素(如果有)和任何后续元素移到右边(将一个元素添加到它们的索引中)。
* 
* @param 要插入的索引位置
* @param 要添加的元素
* @throws 如果 index 大于集合长度 或者小于 0 则抛出角标越界 IndexOutOfBoundsException 异常
*/
public void add(int index, E element) {
   // 检查角标是否越界
   rangeCheckForAdd(index);
    // 扩容检查
   ensureCapacityInternal(size + 1);      
   //调用 native 方法新型数组拷贝
   System.arraycopy(elementData, index, elementData, 
                    index + 1,size - index);
    // 添加新元素
   elementData[index] = element;
   size++;
}

private void rangeCheckForAdd(int index) {
   if (index > size || index < 0)
       throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

### 3.4 在数组末尾批量添加元素

```java
public boolean addAll(Collection<? extends E> c) {
        // 调用 c.toArray 将集合转化数组
        Object[] a = c.toArray();
        // 要添加的元素的个数
        int numNew = a.length;
        //扩容检查以及扩容
        ensureCapacityInternal(size + numNew);  // Increments modCount
        //将参数集合中的元素添加到原来数组 [size，size + numNew -1] 的角标位置上。
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        //与单一添加的 add 方法不同的是批量添加有返回值，如果 numNew == 0 表示没有要添加的元素则需要返回 false 
        return numNew != 0;
}
```

### 3.5 在数值指定交标位置批量添加元素

```java
public boolean addAll(int index, Collection<? extends E> c) {
        //同样检查要插入的位置是否会导致角标越界
        rangeCheckForAdd(index);
        
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew); 
        //这里做了判断，如果要numMoved > 0 代表插入的位置在集合中间位置，和在 numMoved == 0最后位置 则表示要在数组末尾添加 如果 < 0  rangeCheckForAdd 就跑出了角标越界
        int numMoved = size - index;
        
    	if (numMoved > 0) //将指定插入点后面的元素后移numMoved位
            System.arraycopy(elementData, index, elementData, 								index + numNew, numMoved);
        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }
    

```

 与单一添加的 add 方法不同的是批量添加有返回值，如果 numNew == 0 表示没有要添加的元素则需要返回 false

## 4 ArrayList删除元素

### 4.1 根据角标移除元素

```java
/**
* 将任何后续元素移到左边(从它们的索引中减去一个)。
*/
public E remove(int index) {
   //检查 index 是否 >= size
   rangeCheck(index);

   modCount++;
   //index 位置的元素 
   E oldValue = elementData(index);
    // 需要移动的元素个数
   int numMoved = size - index - 1;
   if (numMoved > 0)
        //采用拷贝赋值的方法将 index 之后所有的元素 向前移动一个位置
       System.arraycopy(elementData, index+1, elementData, 								index, numMoved);
   // 将 element 末尾的元素位置设为 null                 
   elementData[--size] = null; // clear to let GC do its work
    // 返回 index 位置的元素 
   return oldValue;
}

// 比较要移除的角标位置和当前 elementData 中元素的个数
private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
}
```

根据角标移除元素的方法源码如上所示，值得注意的地方是：

`rangeCheck` 和 `rangeCheckForAdd` 方法不同 ，`rangeCheck` 只检查了 index是否**大于等于** `size`，因为我们知道 `size` 为`elementData` 已存储数据的个数，我们只能移除 `elementData` 数组中 `[0 , size -1]` 的元素，否则应该抛出角标越界。

但是为什么没有 和 `rangeCheckForAdd` 一样检查小于0的角标呢，是不是`remove（-1）` 不会抛异常呢？ 其实不是的，因为 rangeCheck(index); 后我们去调用 `elementData(index)`的时候也会抛出 `IndexOutOfBoundsException` 的异常，这是数组本身抛出的，不是 ArrayList 抛出的。那为什么要检查`>= size` 呢？ 数组本身不也会检查么？ 哈哈.. 细心的同学肯定知道 `elementData.length` 并不一定等于 `size`，比如：

```java
   ArrayList<String> testRemove = new ArrayList<>(10);

   testRemove.add("1");
   testRemove.add("2");
    // java.lang.IndexOutOfBoundsException: Index: 2, Size: 2
   String remove = testRemove.remove(2);
    
   System.out.println("remove = " + remove + "");
```

new ArrayList<>(10) 表示 `elementData` 初始容量为10，所以`elementData.length = 10` 而我们只给集合添加了两个元素所以 `size = 2` 这也就是为啥要 `rangeCheck` 的原因了。

### 4.2 移除指定元素

```java
/**
* 删除指定元素，如果它存在则返回true，如果不存在返回 false。
* 更准确地说是删除集合中第一个出现o元素位置的元素 ，
* 也就是说只会删除一个，并且如果有重复的话，只会删除第一个次出现的位置。
* 如果存放的是对象的话，需要重写对象的equal方法，要不然就是比较的是引用值了
*/
public boolean remove(Object o) {
    // 如果元素为空则只需判断 == 也就是内存地址
   if (o == null) {
       for (int index = 0; index < size; index++)
           if (elementData[index] == null) {
                //得到第一个等于 null 的元素角标并移除该元素 返回 ture
               fastRemove(index);
               return true;
           }
   } else {
        // 如果元素不为空则需要用 equals 判断。
       for (int index = 0; index < size; index++)
           if (o.equals(elementData[index])) {
                //得到第一个等于 o 的元素角标并移除该元素 返回 ture
               fastRemove(index);
               return true;
           }
   }
   return false;
}

//移除元素的逻辑和 remve(Index)一样 
private void fastRemove(int index) {
   modCount++;
   int numMoved = size - index - 1;
   if (numMoved > 0)
       System.arraycopy(elementData, index+1, elementData, index,
                        numMoved);
   elementData[--size] = null; // clear to let GC do its work
}
```

由上边代码可以看出来，移除元素和移除指定角标元素一样最终都是 通过 `System.arraycopy` 将 index 之后的元素前移一位，并释放原来位于 size 位置的元素。

还可以看出，如果数组中有指定多个与 o 相同的元素只会移除角标最小的那个，并且 null 和 非null 的时候判断方法不一样。

### 4.3 批量移除/保留 removeAll/retainAll

ArrayList 提供了 `removeAll/retainAll` 操作，这两个操作分别是 批量删除与参数集合中共同享有的元素 和 批量删除与参数集合中不共同享有的元。**注意：这将修改原来的List**

```java
/** 批量删除与参数集合中共同享有的元素*/
 public boolean removeAll(Collection<?> c) {
        //判空 如果为空则抛出 NullPointerException 异常 Objects 的方法
        Objects.requireNonNull(c);
        return batchRemove(c, false);
 }
 
 /** 只保留与 c 中元素相同的元素相同的元素*/
public boolean retainAll(Collection<?> c) {
   Objects.requireNonNull(c);
   return batchRemove(c, true);
}
 
 /** 批量删除的指定方法 */
private boolean batchRemove(Collection<?> c, boolean complement) {
   
   final Object[] elementData = this.elementData;
    // r w 两个角标 r 为 elementData 中元素的索引 
    // w 为删除元素后集合的长度 
   int r = 0, w = 0;
   boolean modified = false;
   try {
       for (; r < size; r++)
            // 如果 c 当前集合中不包含当前元素，那么则保留
           if (c.contains(elementData[r]) == complement)
               elementData[w++] = elementData[r];
   } finally {
       // c.contains（o）可能会抛出异常，如果抛出异常后 r!=size 则将 r 之后的元素不在比较直接放入数组
       if (r != size) {
           System.arraycopy(elementData, r,
                            elementData, w,
                            size - r);
          // w 加上剩余元素的长度
           w += size - r;
       }
        // 如果集合移除过元素，则需要将 w 之后的元素设置为 null 释放内存
       if (w != size) {
           // clear to let GC do its work
           for (int i = w; i < size; i++)
               elementData[i] = null;
           modCount += size - w;
           size = w;
           modified = true;
       }
   }
   //返回是否成功移除过元素，哪怕一个
   return modified;
}
```

## 5 ArrayList的改查

### 5.1 修改指定角标位置的元素

```java
public E set(int index, E element) {
    //角标越界检查,判断index是否小于size
   rangeCheck(index);
 //下标取数据注意这里不是elementData[index] 而是 elementData(index) 方法
   E oldValue = elementData(index);
   //将 index 位置设置为新的元素
   elementData[index] = element;
   // 返回之前在 index 位置的元素
   return oldValue;
}

E elementData(int index) {
    return (E) elementData[index];
}
```

 ### 5.2 查询指定角标的元素

```java
public E get(int index) {
    //越界检查 判断index是否小于size
    rangeCheck(index);
    //下标取数据注意这里不是elementData[index] 而是 elementData(index) 方法
    return elementData(index); 
}
```

### 5.3 查询指定元素的角标或者集合是否包含某个元素

```java
//集合中是否包含元素 indexOf 返回 -1 表示不包含 return false 否则返回 true
public boolean contains(Object o) {
   return indexOf(o) >= 0;
}

/**
* 返回集合中第一个与 o 元素相等的元素角标，返回 -1 表示集合中不存在这个元素
* 这里还做了空元素直接判断 == 的操作
*/
public int indexOf(Object o) {
   if (o == null) {
       for (int i = 0; i < size; i++)
           if (elementData[i]==null)
               return i;
   } else {
       for (int i = 0; i < size; i++)
           if (o.equals(elementData[i]))
               return i;
   }
   return -1;
}
    
 /** 
  * 从 elementData 末尾开始遍历遍历数组，所以返回的是集合中最后一个与 o 相等的元素的角标
  */
public int lastIndexOf(Object o) {
   if (o == null) {
       for (int i = size-1; i >= 0; i--)
           if (elementData[i]==null)
               return i;
   } else {
       for (int i = size-1; i >= 0; i--)
           if (o.equals(elementData[i]))
               return i;
   }
   return -1;
}
```

 

## 6 ArrayList集合的toArray方法

其实 `Object[] toArray();` 方法，以及其重载函数 `<T> T[] toArray(T[] a);` 是接口 `Collection` 的方法，ArrayList 实现了这两个方法，很少见ArrayList 源码分析的文章分析这两个方法，顾名思义这两个方法的是用来，将一个集合转为数组的方法，那么两者的不同之处是，后者可以指定数组的类型，前者返回为一个 Object[] 超类数组。那么我们具体下源码实现：

```java
public Object[] toArray() {
   return Arrays.copyOf(elementData, size);
}

@SuppressWarnings("unchecked")
public <T> T[] toArray(T[] a) {
   if (a.length < size)
       // Make a new array of a's runtime type, but my contents:
       return (T[]) Arrays.copyOf(elementData, size, a.getClass());
   System.arraycopy(elementData, 0, a, 0, size);
   if (a.length > size)
       a[size] = null;
   return a;
}
```

我们可以传入一个指定类型的标志数组作为参数，`toArray(T[] a)` 方法最终会返回这个类型的包含集合元素的新数组。但是源码判断了 ：

1. 如果 `a.length < size` 即当前集合元素的个数比参数 a 数组元素的小的时候将和 `toArray()` 一样返回一个新的数组。
2. 如果 `a.length == size` 将不会产生新的数组直接将集合中的元素调用 `System.arraycopy` 方法将元素复制到参数数组中，返回 a。
3. `a.length > size` 也不会产生新的数组,但是值得注意的是 `a[size] = null;` 这一句改变了原数组中 index = size 位置的元素，被重新设置为 null 了。

 

##  7 subList注意点

```java
public List<E> subList(int fromIndex, int toIndex) {
    subListRangeCheck(fromIndex, toIndex, size);
    return new SubList(this, 0, fromIndex, toIndex);
}

static void subListRangeCheck(int fromIndex, int toIndex, int size) {
    if (fromIndex < 0)
        throw new IndexOutOfBoundsException("fromIndex = " + fromIndex);
    //这个点要注意， 当toIndex>size会抛出异常，而不是自认为的截取[0,size)区间的元素
    if (toIndex > size)
        throw new IndexOutOfBoundsException("toIndex = " + toIndex);
    if (fromIndex > toIndex)
        throw new IllegalArgumentException("fromIndex(" + fromIndex +
                                           ") > toIndex(" + toIndex + ")");
}
```

```java
private class SubList extends AbstractList<E> implements RandomAccess {
    private final AbstractList<E> parent;
    private final int parentOffset;
    private final int offset;
    int size;

    SubList(AbstractList<E> parent,
            int offset, int fromIndex, int toIndex) {
        this.parent = parent;  //由这可以看出，subList方法截取的子List还是原来的List的一份拷贝，对这个子List做任务操作都会反映到原List上。
        this.parentOffset = fromIndex;
        this.offset = offset + fromIndex;
        this.size = toIndex - fromIndex;
        this.modCount = ArrayList.this.modCount;
    }
```

当我们使用子集合tempList进行元素的修改操作时，会影响原有的list集合。所以在使用subList方法时，一定要想清楚，是否需要对子集合进行修改元素而不影响原有的list集合

如果**需要对子集合的元素进行修改操作而不需要影响原集合时**，我们可以使用以下方法进行处理：

> **List<Object> tempList = new ArrayList<Object>(lists.subList(2, lists.size()));**   

------



## 8 错误操作导致 ConcurrentModificationException 异常

`ConcurrentModificationException`是指因为迭代器调用 `checkForComodification` 方法比较 `modCount` 和 `expectedModCount` 方法大小的时候抛出异常。我们在分析 ArrayList 的时候在每次对集合进行修改， 即有 add 和 remove 操作的时候每次都会对 `modCount ++`。

modCount 这个变量主要用来记录 `ArrayList` 被修改的次数，那么为什么要记录这个次数呢？是为了防止多线程对同一集合进行修改产生错误，记录了这个变量，在对 ArrayList 进行迭代的过程中我们能很快的发现这个变量是否被修改过，如果被修改了 `ConcurrentModificationException` 将会产生。下面我们来看下例子，这个例子并不是在多线程下的，而是因为我们在同一线程中对 list 进行了错误操作导致的：

```java

Iterator<String> iterator = lists.iterator();

while (iterator.hasNext()) {
  String str = iterator.next();
  if (str.equals("3333")) {
      list.remove(index);//操作1： 注意是 list2.remove 操作
      //iterator.remove()；/操作2 注意是 iterator.remove 操作
  }
}
//操作1： Exception in thread "main" java.util.ConcurrentModificationException
//操作2： 正常打印
System.out.println(list2);
```

我们对操作1，2分别运行程序，可以看到，操作1很快就抛出了 `java.util.ConcurrentModificationException` 异常，操作2 则顺利运行出正常结果，如果对 `modCount` 注意了的话，我们很容易理解，`list.remove(index)` 操作会修改`List` 的 `modCount`，而 `iterator.next()` 内部每次会检验 `expectedModCount != modCount`，所以当我们使用 `list.remove` 下一次再调用 `iterator.next()` 就会报错了，而`iterator.remove`为什么是安全的呢？因为其操作内部会在调用 `list.remove` 后重新将新的 `modCount` 赋值给 `expectedModCount`。所以我们直接调用 list.remove 操作是错误的。

让我们来看看下面这道面试题：

```java
ArrayList<String> list = new ArrayList<String>();
for (int i = 0; i < 10; i++) {
  list.add("sh" + i);
}

for (int i = 0; list.iterator().hasNext(); i++) {
  list.remove(i);
  System.out.println("秘密" + list.get(i));
}
```

相信大家肯定知道这样操作是会产生错误的，但是最终会抛出角标越界还是`ConcurrentModificationException`呢？

其实这里会抛出角标越界异常，为什么呢，因为 for 循环的条件 `list.iterator().hasNext()`，我们知道 `list.iterator()` 将会new 一个新的 iterator 对象，而在 new 的过程中我们将 每次 `list.remove` 后的 `modCount` 赋值给了新的 `iterator`的 `expectedModCount`，所以不会抛出 `ConcurrentModificationException` 异常，而 `hasNext` 内部只判断了 size 是否等于 `cursor != size` 当我们删除了一半元素以后，size 变成了 5 而新的 `list.iterator()` 的 cursor 等于 0 ，`0!=5` for 循环继续，那么当执行到 `list.remove（5）`的时候就会抛出角标越界了。

## 9 总结

1. ArrayList 底层是一个动态扩容的数组结构,每次扩容需要增加1.5倍的容量

2. ArrayList 扩容底层是通过 `Arrays.CopyOf` 和 `System.arraycopy` 来实现的。每次都会产生新的数组，和数组中内容的拷贝，所以会耗费性能，所以在多增删的操作的情况可优先考虑 LinkList 而不是 ArrayList。

3. ArrayList 的 toArray 方法重载方法的使用。

4. 允许存放（不止一个） null 元素，

5. 允许存放重复数据，存储顺序按照元素的添加顺序

6. ArrayList 并不是一个线程安全的集合。如果集合的增删操作需要保证线程的安全性，可以考虑使用 `CopyOnWriteArrayList` 或者使`collections.synchronizedList(List l)`函数返回一个线程安全的ArrayList类.

7. 当ArrayList集合中存放的是对象的时候，调用index(),remove(),contain()方法时，需要重写对象的equal方法。

   

















