# HashMap源码剖析（1.7）

[TOC]

## HashMap介绍

```java
public class HashMap<K,V>
       extends AbstractMap<K,V>
       implements Map<K,V>, Cloneable, Serializable
```

可以看到`HashMap`继承了

- 标记接口[Cloneable](http://docs.oracle.com/javase/7/docs/api/index.html?java/lang/Cloneable.html)，用于表明`HashMap`对象会重写`java.lang.Object#clone()`方法，HashMap实现的是浅拷贝（shallow copy）。
- 标记接口[Serializable](http://docs.oracle.com/javase/7/docs/api/index.html?java/io/Serializable.html)，用于表明`HashMap`对象可以被序列化

比较有意思的是，`HashMap`同时继承了抽象类`AbstractMap`与接口`Map`，因为抽象类`AbstractMap`的签名为:

```java
public abstract class AbstractMap<K,V> implements Map<K,V>
```

[Stack Overfloooow](http://stackoverflow.com/questions/14062286/java-why-does-weakhashmap-implement-map-whereas-it-is-already-implemented-by-ab)上解释到：

> 在语法层面实现接口`Map`是多余的，这么做仅仅是为了让阅读代码的人明确知道`HashMap`是属于`Map`体系的，起到了文档的作用

`AbstractMap`相当于个辅助类，`Map`的一些操作这里面已经提供了默认实现，后面具体的子类如果没有特殊行为，可直接使用`AbstractMap`提供的实现。

[Cloneable](http://docs.oracle.com/javase/7/docs/api/index.html?java/lang/Cloneable.html)接口

```
It's evil, don't use it.
```

`Cloneable`这个接口设计的非常不好，最致命的一点是它里面竟然没有`clone`方法，也就是说我们自己写的类完全可以实现这个接口的同时不重写`clone`方法。

关于`Cloneable`的不足，大家可以去看看《Effective Java》一书的作者[给出的理由](http://www.artima.com/intv/bloch13.html)，在所给链接的文章里，Josh Bloch也会讲如何实现深拷贝比较好，我这里就不在赘述了

`Map`虽然并不是`Collection`，但是它提供了三种“集合视角”（collection views），与下面三个方法一一对应：

- `Set<K> keySet()`，提供key的集合视角
- `Collection<V> values()`，提供value的集合视角
- `Set<Map.Entry<K, V>> entrySet()`，提供key-value序对的集合视角，这里用内部类`Map.Entry`表示序对

## 设计理念

`HashMap`是一种基于[哈希表（hash table）](https://en.wikipedia.org/wiki/Hash_table)实现的map，哈希表（也叫关联数组）一种通用的数据结构，大多数的现代语言都原生支持，其概念也比较简单：`key经过hash函数作用后得到一个槽（buckets或slots）的索引（index），槽中保存着我们想要获取的值`

![hashmap.png](https://i.loli.net/2018/08/05/5b66c7b178732.png)

## HashMap的一些特点

- 线程非安全，并且允许key与value都为null值，hashMap中key为null,则算出的hash值为0，`HashTable`与之相反，为线程安全，key与value都不允许null值。
- 不保证其内部元素的顺序，而且随着时间的推移，同一元素的位置也可能改变（resize的情况）
- put、get操作的时间复杂度为O(1)。
- 遍历其集合视角的时间复杂度与其容量（capacity，槽的个数）和现有元素的大小（entry的个数）成正比，所以如果遍历的性能要求很高，不要把capactiy设置的过高或把平衡因子（load factor，当entry数大于capacity*loadFactor时，会进行resize，reside会导致key进行rehash）设置的过低。
- 由于HashMap是线程非安全的，这也就是意味着如果多个线程同时对一hashmap的集合试图做迭代时有结构的上改变（添加、删除entry，只改变entry的value的值不算结构改变），那么会报[ConcurrentModificationException](http://docs.oracle.com/javase/7/docs/api/java/util/ConcurrentModificationException.html)，专业术语叫`fail-fast`，尽早报错对于多线程程序来说是很有必要的。
- `Map m = Collections.synchronizedMap(new HashMap(...));`通过这种方式可以得到一个线程安全的map。

## 源码剖析

```java
//容量（capacity）： HashMap中数组的长度
// a. 容量范围：必须是2的幂 & <最大容量（2的30次方）
// b. 初始容量 = 哈希表创建时的容量
// 默认容量 = 16 = 1<<4 
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; 

// 最大容量 =  2的30次方（若用户传入的容量过大，将被这个值替换）
static final int MAXIMUM_CAPACITY = 1 << 30;
//加载因子
final float loadFactor;
//默认加载因子=0.75
static final float DEFAULT_LOAD_FACTOR = 0.75f;

//扩容阈值（threshold）：当哈希表的大小 ≥ 扩容阈值时，就会扩容哈希表（即扩充HashMap的容量） 
//a.扩容 = 对哈希表进行resize操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数
//b.扩容阈值 = 容量 x 加载因子
int threshold;

// 存储数据的Entry类型数组,用来保存添加进来的Entry对象，下文详细介绍
transient Entry<K,V>[] table = (Entry<K,V>[]) EMPTY_TABLE;  
// HashMap的大小，即 HashMap中存储的键值对的数量
transient int size;
```

从代码上可以看到，容量与加载因子都有个默认值，并且容量有个最大值。这里有个比较奇怪的问题是：容量必须是2的n次幂，这是为啥呢？解答这个问题，需要了解HashMap中哈希函数的设计原理。

### 哈希函数的设计原理

```java
 final int hash(Object k) {
     int h = hashSeed;
     if (0 != h && k instanceof String) {
         return sun.misc.Hashing.stringHash32((String) k);
     }
     h ^= k.hashCode();
     h ^= (h >>> 20) ^ (h >>> 12);
     return h ^ (h >>> 7) ^ (h >>> 4);
 }

 static int indexFor(int h, int length) {
     return h & (length-1);
 }
```

在哈希表容量（也就是buckets或slots大小）为length的情况下，为了使每个key都能在冲突最小的情况下映射到`[0,length)`（注意是左闭右开区间）的索引（index）内，一般有两种做法：

1. 让length为素数，然后用`hashCode(key) mod length`的方法得到索引
2. 让length为2的指数倍，然后用`hashCode(key) & (length-1)`的方法得到索引

[HashTable](http://docs.oracle.com/javase/7/docs/api/index.html?java/util/Hashtable.html)用的是方法1，`HashMap`用的是方法2。

重点说说方法2的情况，方法2其实也比较好理解：

> 因为length为2的指数倍，所以`length-1`所对应的二进制位都为1，然后在与`hashCode(key)`做与运算，即可得到`[0,length)`内的索引,也就达到求模运算一样的结果。但是位运算效率更高。

但是这里有个问题，如果`hashCode(key)`的大于`length`的值，而且`hashCode(key)`的二进制位的低位变化不大，那么冲突就会很多，举个例子：

> Java中对象的哈希值都32位整数，而HashMap默认大小为16，那么有两个对象那么的哈希值分别为：`0xABAB0000`与`0xBABA0000`，它们的后几位都是一样，那么与16异或后得到结果应该也是一样的，也就是产生了冲突。

造成冲突的原因关键在于16限制了只能用低位来计算，高位直接舍弃了，所以我们需要额外的哈希函数而不只是简单的对象的`hashCode`方法了。
具体来说，就是HashMap中`hash`函数干的事了

> 首先有个随机的hashSeed，来降低冲突发生的几率
>
> 然后如果是字符串，用了`sun.misc.Hashing.stringHash32((String) k);`来获取索引值
>
> 最后，通过一系列无符号右移操作，来把高位与低位进行异或操作，来降低冲突发生的几率

### 构造函数

```java
//空的构造函数使用默认的容量(capacity=16)和负载因子（loadFactor=0.75)
public HashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR);
}

//指定容量
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

//指定容量和负载因子
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " +
                                           initialCapacity);
     // HashMap的最大容量只能是MAXIMUM_CAPACITY，哪怕传入的 > 最大容量
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " +
                                           loadFactor);

    
    this.loadFactor = loadFactor;
     // 注：此处不是真正的阈值，是为了扩展table，该阈值后面会重新计算，下面会详细讲解 
    threshold = initialCapacity;
    init(); //一个空方法用于未来子对象的扩展
}
```

注： 

1. 此处仅用于接收初始容量大小（`capacity`）、加载因子(`Load factor`)，但仍无真正初始化哈希表，即初始化存储数组`table`
2. 此处先给出结论：**真正初始化哈希表（初始化存储数组table）是在第1次添加键值对时，即第1次调用put（）时。下面会详细说明**

### HashMap.Entry

HashMap中存放的是HashMap.Entry对象，它继承自Map.Entry，其比较重要的是构造函数

```java
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;      //键
    V value;          //值
    Entry<K,V> next;  //指向下一个节点
    int hash;       // hash值

    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }
     /** 
     * equals（）
     * 作用：判断2个Entry是否相等，必须key和value都相等，（ 兼容基础类型（==）和对象类型（equals） ）才返回true  
     */ 
      public final boolean equals(Object o) {  
        if (!(o instanceof Map.Entry))  
            return false;  
        Map.Entry e = (Map.Entry)o;  
        Object k1 = getKey();  
        Object k2 = e.getKey();  
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {  
            Object v1 = getValue();  
            Object v2 = e.getValue();  
            if (v1 == v2 || (v1 != null && v1.equals(v2)))  
                return true;  
        }  
        return false;  
    }  
    
    // setter, getter, equals, toString 方法省略
    public final int hashCode() {
        //用key的hash值与上value的hash值作为Entry的hash值
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }
  
    /** 
     * 当向HashMap中添加元素时，即调用put(k,v)时， 
     * 对已经在HashMap中k位置进行v的覆盖时，会调用此方法 
     * 此处没做任何处理 
     */  
    void recordAccess(HashMap<K,V> m) {
    }

   
    /** 
     * 当从HashMap中删除了一个Entry时，会调用该函数 
     * 此处没做任何处理 
     */ 
    void recordRemoval(HashMap<K,V> m) {
    }
}
```

可以看到，Entry实现了单向链表的功能，用`next`成员变量来级连起来。相同索引值的Entry，会以单向链表的形式存在，最终效果图如下：

![选区_282.png](https://i.loli.net/2018/08/05/5b66d8865f345.png)

### get操作

get操作相比put操作简单点，这里先介绍get操作：

```java
public V get(Object key) {
    //单独处理key为null的情况
    if (key == null)
        return getForNullKey();
    Entry<K,V> entry = getEntry(key);

    return null == entry ? null : entry.getValue();
}
private V getForNullKey() {
    if (size == 0) {
        return null;
    }
    //key为null的Entry用于放在table[0]中，但是在table[0]冲突链中的Entry的key不一定为null
    //所以需要遍历冲突链，查找key是否存在
    for (Entry<K,V> e = table[0]; e != null; e = e.next) {
        if (e.key == null)
            return e.value;
    }
    return null;
}
final Entry<K,V> getEntry(Object key) {
    if (size == 0) {
        return null;
    }

    int hash = (key == null) ? 0 : hash(key);
    //首先定位到索引在table中的位置
    //然后遍历冲突链，查找key是否存在
    for (Entry<K,V> e = table[indexFor(hash, table.length)];
         e != null;
         e = e.next) {
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k))))
            return e;
    }
    return null;
}
```

### put操作

```java
public V put(K key, V value) {
    //若 哈希表未初始化（table为空）  使用构造函数设置的阀值（也就是初始容量）初始化数组table
    if (table == EMPTY_TABLE) {
        inflateTable(threshold);                   //分析一：初始化哈希表
    }
    //判断key是否为空值null, 如果key==null, 则将该entry存放到数组table中的第一个位置，也就是table[0]中的单链表中
    if (key == null)
      return putForNullKey(value);                //分析二： key==null，存放
    //根据hash值算出最终key对应存在数组table中的位置
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    //这里的循环是关键
    //当新增的key所对应的索引i，对应table[i]中已经有值时，进入循环体
    for (Entry<K,V> e = table[i]; e != null; e = e.next) {
        Object k;
        //判断是否存在本次插入的key，如果存在用本次的value替换之前oldValue，相当于update操作
        //并返回之前的oldValue
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue; //返回旧的value
        }
    }
    //如果本次新增key之前不存在于HashMap中，modCount加1，说明结构改变了
    modCount++;
    //若果key不存在， 则将“key-value”添加到table中
    addEntry(hash, key, value, i);                //分析三： key-value存放
    return null;
}
```

#### 分析一：初始化哈希表

````java
	 /**
     * 函数使用原型
     */
      if (table == EMPTY_TABLE) { 
        inflateTable(threshold); 
    }  	

	/**
     * 源码分析：inflateTable(threshold); 
     */
     private void inflateTable(int toSize) {  

    // 1. 将传入的容量大小转化为：>传入容量大小的最小的2的次幂
    // 即如果传入的是容量大小是19，那么转化后，初始化容量大小为32（即2的5次幂）
    int capacity = roundUpToPowerOf2(toSize);  

    // 2. 重新计算阈值 threshold = 容量 * 加载因子  
    threshold = (int) Math.min(capacity * loadFactor, MAXIMUM_CAPACITY + 1);  

    // 3. 使用计算后的初始容量（已经是2的次幂） 初始化数组table（作为数组长度）
    // 即 哈希表的容量大小 = 数组大小（长度）
    table = new Entry[capacity]; //用该容量初始化table  

    initHashSeedAsNeeded(capacity);  
}  

     private static int roundUpToPowerOf2(int number) {  

       //若 容量超过了最大值，初始化容量设置为最大值 ；否则，设置为：>传入容量大小的最小的2的次幂
       return number >= MAXIMUM_CAPACITY  ? 
            MAXIMUM_CAPACITY  : (number > 1) ? Integer.highestOneBit((number - 1) << 1) : 1;  
````

- 注：**看这代码知道：真正初始化哈希表（初始化存储数组table）是在第1次添加键值对时，即第1次调用put（）时，**

#### 分析二： key==null,存放

```java
 	/**
     * 函数使用原型
     */
      if (key == null)
           return putForNullKey(value);

    /**
     * 源码分析：putForNullKey(value)
     */
      private V putForNullKey(V value) {  
        // 遍历以table[0]为首的链表，寻找是否存在key==null 对应的键值对
        // 1. 若有：则用新value 替换 旧value；同时返回旧的value值
        for (Entry<K,V> e = table[0]; e != null; e = e.next) {  
          if (e.key == null) {   
            V oldValue = e.value;  
            e.value = value;  
            e.recordAccess(this);  
            return oldValue;  
        }  
    }  
    modCount++;  

    // 2 .若无key==null的键，那么调用addEntry（），将空键 & 对应的值封装到Entry中，并放到table[0]中
    addEntry(0, null, value, 0); 
    // 注：
    // a. addEntry（）的第1个参数 = hash值 = 传入0
    // b. 即 说明：当key = null时，也有hash值 = 0，所以HashMap的key 可为null
    // c. 对比HashTable，由于HashTable对key直接hashCode（），若key为null时，会抛出异常，所以HashTable的key不可为null
    // d. 此处只需知道是将 key-value 添加到HashMap中即可，关于addEntry（）的源码分析将等到下面再详细说明，
    return null;  

}     
```

#### 分析三： key-value存放

```java
	/**
     * 源码分析：addEntry(hash, key, value, i)
     * 作用：添加键值对（Entry ）到 HashMap中
     */
void addEntry(int hash, K key, V value, int bucketIndex) {
    //插入前，先判断容量是否足够，不够的话，扩容
    if ((size >= threshold) && (null != table[bucketIndex])) {
        //大小扩容到原来的2倍
        resize(2 * table.length);
        //重新计算该key对应的hash值
        hash = (null != key) ? hash(key) : 0;
        //重新计算该key对应hash值的存储数组的下标位置
        bucketIndex = indexFor(hash, table.length);
    }

    //容量足够，用key,value值构建一个entry元素，插入到hashMap中
    createEntry(hash, key, value, bucketIndex);
}
void createEntry(int hash, K key, V value, int bucketIndex) {
    //首先得到该索引处的冲突链Entries，第一次插入bucketIndex位置时冲突链为null，也就是e为null
    Entry<K,V> e = table[bucketIndex];
    //然后把新的Entry添加到冲突链的开头，然后这个元素的next域指向e。也就是说，后插入的反而在前面
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}

//下面看看HashMap是如何进行扩容。
void resize(int newCapacity) {
    Entry[] oldTable = table;
    int oldCapacity = oldTable.length;
    //如果已经达到最大容量，那么就直接返回
    if (oldCapacity == MAXIMUM_CAPACITY) {
        threshold = Integer.MAX_VALUE;
        return;
    }

    Entry[] newTable = new Entry[newCapacity];
    //initHashSeedAsNeeded(newCapacity)的返回值决定了是否需要重新计算Entry的hash值
    transfer(newTable, initHashSeedAsNeeded(newCapacity));
    table = newTable;
    //重新设置阀值
    threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
}

/**
 * 将旧数组上的数据（键值对）
 */
void transfer(Entry[] newTable, boolean rehash) {
    int newCapacity = newTable.length;
    //遍历当前的table，将里面的元素添加到新的newTable中
    for (Entry<K,V> e : table) {
        while(null != e) {
            Entry<K,V> next = e.next;
            if (rehash) {
                e.hash = null == e.key ? 0 : hash(e.key);
            }
            int i = indexFor(e.hash, newCapacity);
            //头插法，插入到扩容的hashmap中
            e.next = newTable[i];
            newTable[i] = e;
            e = next;
        }
    }
}
```

### putAll操作

```java
	/**
   * 函数：putAll(Map<? extends K, ? extends V> m)
   * 作用：将指定Map中的键值对 复制到 此Map中
   * 原理：类似Put函数
   */ 

    public void putAll(Map<? extends K, ? extends V> m) {  
    // 1. 统计需复制多少个键值对  
    int numKeysToBeAdded = m.size();  
    if (numKeysToBeAdded == 0)  
        return; 

    // 2. 若table还没初始化，先用刚刚统计的复制数去初始化table  
    if (table == EMPTY_TABLE) {  
        inflateTable((int) Math.max(numKeysToBeAdded * loadFactor, threshold));  
    }  

    // 3. 若需复制的数目 > 阈值，则需先扩容 
    if (numKeysToBeAdded > threshold) {  
        int targetCapacity = (int)(numKeysToBeAdded / loadFactor + 1);  
        if (targetCapacity > MAXIMUM_CAPACITY)  
            targetCapacity = MAXIMUM_CAPACITY;  
        int newCapacity = table.length;  
        while (newCapacity < targetCapacity)  
            newCapacity <<= 1;  
        if (newCapacity > table.length)  
            resize(newCapacity);  
    }  
    // 4. 开始复制（实际上不断调用Put函数插入）  
    for (Map.Entry<? extends K, ? extends V> e : m.entrySet())  
        put(e.getKey(), e.getValue());
}  
```

### remove操作

```java
public V remove(Object key) {
    Entry<K,V> e = removeEntryForKey(key);
    //可以看到删除的key如果存在，就返回其所对应的value
    return (e == null ? null : e.value);
}
final Entry<K,V> removeEntryForKey(Object key) {
    if (size == 0) {
        return null;
    }
    int hash = (key == null) ? 0 : hash(key);
    int i = indexFor(hash, table.length);
    //这里用了两个Entry对象，相当于两个指针，为的是防治冲突链发生断裂的情况
    //这里的思路就是一般的单向链表的删除思路
    Entry<K,V> prev = table[i];
    Entry<K,V> e = prev;

    //当table[i]中存在冲突链时，开始遍历里面的元素
    while (e != null) {
        Entry<K,V> next = e.next;
        Object k;
        if (e.hash == hash &&
            ((k = e.key) == key || (key != null && key.equals(k)))) {
            modCount++;
            size--;
            if (prev == e) //当要删除的节点是链表中的头节点
                table[i] = next;
            else
                prev.next = next;
            e.recordRemoval(this);
            return e;
        }
        prev = e;
        e = next;
    }

    return e;
}
```

到现在为止，HashMap的增删改查都介绍完了。 一般而言，认为HashMap的这四种操作时间复杂度为O(1)，因为它hash函数性质较好，保证了冲突发生的几率较小。

### containKey操作

```java
 /**
   * 函数：containsKey(Object key)
   * 作用：判断是否存在该键的键值对；是 则返回true
   * 原理：调用getEntry（），判断是否为Null
   */
   public boolean containsKey(Object key) {  
    return getEntry(key) != null; 
} 

final Entry<K,V> getEntry(Object key) {  
    if (size == 0) {  
        return null;  
    }  

    // 1. 根据key值，通过hash（）计算出对应的hash值
    int hash = (key == null) ? 0 : hash(key);  

    // 2. 根据hash值计算出对应的数组下标
    // 3. 遍历 以该数组下标的数组元素为头结点的链表所有节点，寻找该key对应的值
    for (Entry<K,V> e = table[indexFor(hash, table.length)];  e != null;  e = e.next) {  

        Object k;  
        // 若 hash值 & key 相等，则证明该Entry = 我们要的键值对
        // 通过equals（）判断key是否相等
        if (e.hash == hash &&  
            ((k = e.key) == key || (key != null && key.equals(k))))  
            return e;  
    }  
    return null;  
}
```



## 补充

### 问题一 ：为啥hashMap不是线程安全的

HashMap会进行resize操作，在resize操作的时候会造成线程不安全。下面将举两个可能出现线程不安全的地方。

- 1、put的时候导致的多线程数据不一致。

 这个问题比较好想象，比如有两个线程A和B，首先A希望插入一个key-value对到HashMap中，首先计算记录所要落到的桶的索引坐标，然后获取到该桶里面的链表头结点，也就是运行到createEntry()方法中的Entry<K,V>e=table[bucketIndex]此时线程A的时间片用完了。

而此时线程B被调度得以执行，和线程A一样执行，只不过线程B成功将记录插到了桶里面，假设线程A插入的记录计算出来的桶索引和线程B要插入的记录计算出来的桶索引是一样的。

那么当线程B成功插入之后，线程A再次被调度运行时，此时e的指向是单链表中第二个元素，第一个元素是B插入后的元素。运行 table[bucketIndex] = new Entry<>(hash, key, value, e);如此一来就覆盖了线程B插入的记录，这样线程B插入的记录就凭空消失了，造成了数据不一致的行为。

```java
void createEntry(int hash, K key, V value, int bucketIndex) {
    //首先得到该索引处的冲突链Entries，第一次插入bucketIndex位置时冲突链为null，也就是e为null
    Entry<K,V> e = table[bucketIndex]; //执行完了这句，线程A被切出去。
    
    //然后把新的Entry添加到冲突链的开头，然后这个元素的next域指向e。也就是说，后插入的反而在前面
    table[bucketIndex] = new Entry<>(hash, key, value, e);
    size++;
}
```


- 2.另外一个比较明显的线程不安全的问题是HashMap的put操作可能因为resize而引起死循环（cpu100%），具体分析如下：

  我们假设有两个线程同时需要执行resize操作，我们原来的桶数量为2，记录数量为3，需要resize桶到4，原来的记录分别为：[3,A],[7,B],[5,C]，在原来的map里面，我们发现这三个entry都落到了第二个桶里面。
   假设线程thread1执行到了transfer方法的Entry next = e.next这一句，然后时间片用完了，此时的e = [3,A], next = [7,B]。

  线程thread2被调度执行并且顺利完成了resize操作，需要注意的是，此时的[7,B]的next为[3,A]。此时线程thread1重新被调度运行，此时的thread1持有的引用是已经被thread2 resize之后的结果。线程thread1首先将[3,A]迁移到新的数组上，然后再处理[7,B]，而[7,B]被链接到了[3,A]的后面，处理完[7,B]之后，就需要处理[7,B]的next了啊，而通过thread2的resize之后，[7,B]的next变为了[3,A]，此时，[3,A]和[7,B]形成了环形链表，在get的时候，如果get的key的桶索引和[3,A]和[7,B]一样，那么就会陷入死循环。

  ![选区_283.png](https://i.loli.net/2018/08/05/5b66f462c9926.png)

  如果在取链表的时候从头开始取（现在是从尾部开始取）的话，则可以保证节点之间的顺序，那样就不存在这样的问题了。


```java
void transfer(Entry[] newTable, boolean rehash) {  
        int newCapacity = newTable.length;  
        for (Entry<K,V> e : table) {  
  
            while(null != e) {  
                Entry<K,V> next = e.next;     //这里被切出去了，后会造成死循环 
                if (rehash) {  
                    e.hash = null == e.key ? 0 : hash(e.key);  
                }  
                int i = indexFor(e.hash, newCapacity);   
                e.next = newTable[i];  
                newTable[i] = e;  
                e = next;  
            } 
        }  
    }

```

###  问题二：HashMap 中的 `key`若 `Object`类型， 则需实现哪些方法？

![hashcode.png](https://i.loli.net/2018/08/05/5b66f63ee33ed.png)

### 问题三：HashMap中的key为啥能是null

下面stackoverflow给出一个解释：

https://stackoverflow.com/questions/2945309/what-is-the-use-of-adding-a-null-key-or-value-to-a-hashmap-in-java





```JAVA

```

