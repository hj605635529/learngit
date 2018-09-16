 # HashMap剖析（1.8）

[TOC]

## 一：前言

Java8的HashMap对之前做了较大的优化，其中最重要的一个优化就是桶中的元素不再唯一按照链表组合，也可以使用红黑树进行存储，总之，目标只有一个，那就是在安全和功能性完备的情况下让其速度更快，提升性能。好~下面就开始分析源码。

## 二：HashMap数据结构

![选区_285.png](https://i.loli.net/2018/08/06/5b67224261584.png)

上图很形象的展示了HashMap的数据结构（数组+链表+红黑树），桶中的结构可能是链表，也可能是红黑树，红黑树的引入是为了提高效率。

## 三：HashMap源码解析

```java
public class HashMap<K,V>
         extends AbstractMap<K,V> 
         implements Map<K,V>, Cloneable, Serializable
```

和1.7一样，HashMap依然是extends AbstractMap 实现Map接口。

### 链表节点实现：

```java
/**
 * Node是HashMap内部类，实现了Map.Entry接口，本质是一个键值对
 * 与JDK1.7对比（Entry类)，仅仅是换了一个名字
 */
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;   //哈希值
    final K key;
    V value;
    Node<K,V> next;  //链表的下一个节点

    Node(int hash, K key, V value, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.value = value;
        this.next = next;
    }

    public final K getKey()        { return key; }
    public final V getValue()      { return value; }
    public final String toString() { return key + "=" + value; }

    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    //判断2个Node是否相等，必须key和value都相等，才返回为true.
    public final boolean equals(Object o) {
        if (o == this)
            return true;
        if (o instanceof Map.Entry) {
            Map.Entry<?,?> e = (Map.Entry<?,?>)o;
            if (Objects.equals(key, e.getKey()) &&
                Objects.equals(value, e.getValue()))
                return true;
        }
        return false;
    }
}
```

### 红黑树节点实现：

```java
 /**
  * 红黑树节点 实现类：继承自LinkedHashMap.Entry<K,V>类
  */
  static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {  

    // 属性 = 父节点、左子树、右子树、删除辅助节点 + 颜色
    TreeNode<K,V> parent;  
    TreeNode<K,V> left;   
    TreeNode<K,V> right;
    TreeNode<K,V> prev;   
    boolean red;   

    // 构造函数
    TreeNode(int hash, K key, V val, Node<K,V> next) {  
        super(hash, key, val, next);  
    }  

    // 返回当前节点的根节点  
    final TreeNode<K,V> root() {  
        for (TreeNode<K,V> r = this, p;;) {  
            if ((p = r.parent) == null)  
                return r;  
            r = p;  
        }  
    } 
```

### HashMap中的重要参数

- 在进行真正的源码分析前，先讲解`HashMap`中的重要参数（变量）
- `HashMap`中的主要参数 同 `JDK 1.7` ，即：容量、加载因子、扩容阈值
- 但由于数据结构中引入了 红黑树，故加入了 **与红黑树相关的参数**。具体介绍如下：

```java
/** 
   * 主要参数 同  JDK 1.7 
   * 即：容量、加载因子、扩容阈值（要求、范围均相同）
   */

  // 1. 容量（capacity）： 必须是2的幂 & <最大容量（2的30次方）
  static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; //默认容量16
  // 最大容量 =  2的30次方（若传入的容量过大，将被最大值替换）
  static final int MAXIMUM_CAPACITY = 1 << 30; 

  // 2. 加载因子：HashMap在其容量自动增加前可达到多满的一种尺度 
  final float loadFactor; // 实际加载因子
  static final float DEFAULT_LOAD_FACTOR = 0.75f;//默认加载因子0.75

  // 3. 扩容阈值（threshold）：当哈希表的大小 ≥ 扩容阈值时，就会扩容哈希表（即扩充HashMap的容量） 
  // a. 扩容 = 对哈希表进行resize操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数
  // b. 扩容阈值 = 容量 x 加载因子
  int threshold;

  // 4. 其他
  transient Node<K,V>[] table;  // 存储数据的Node类型 数组，长度 = 2的幂；数组的每个元素是链表的头节点
  transient int size;// HashMap的大小，即 HashMap中存储的键值对的数量


  /** 
   * 与红黑树相关的参数
   */
   // 1. 桶的树化阈值：即 链表转成红黑树的阈值，在存储数据时，当链表长度 > 该值时，则将链表转换成红黑树
   static final int TREEIFY_THRESHOLD = 8; 
   // 2. 桶的链表还原阈值：即 红黑树转为链表的阈值，当在扩容（resize（））时（此时HashMap的数据存储位置会重新计算），在重新计算存储位置后，当原有的红黑树内数量 < 6时，则将 红黑树转换成链表
   static final int UNTREEIFY_THRESHOLD = 6;
   // 3. 最小树形化容量阈值：即 当哈希表中的容量 > 该值时，才允许树形化链表 （即 将链表 转换成红黑树）
   // 否则，若桶内元素太多时，则直接扩容，而不是树形化
   // 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
   static final int MIN_TREEIFY_CAPACITY = 64;
```

- 总结 数据结构和参数方面1.8与1.7的区别：

![hash.png](https://i.loli.net/2018/08/05/5b671afa7bda3.png)

### 构造函数

```java
	/**
     * 构造函数1：默认构造函数（无参）
     * 加载因子 & 容量 = 默认 = 0.75、16
     */
    public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR;
    }

    /**
     * 构造函数2：指定“容量大小”的构造函数
     * 加载因子 = 默认 = 0.75 、容量 = 指定大小
     */
    public HashMap(int initialCapacity) {
        // 实际上是调用指定“容量大小”和“加载因子”的构造函数
        // 只是在传入的加载因子参数 = 默认加载因子
        this(initialCapacity, DEFAULT_LOAD_FACTOR);

    }

    /**
     * 构造函数3：指定“容量大小”和“加载因子”的构造函数
     * 加载因子 & 容量 = 自己指定
     */
    public HashMap(int initialCapacity, float loadFactor) {

        // 指定初始容量必须非负，否则报错  
         if (initialCapacity < 0)  
           throw new IllegalArgumentException("Illegal initial capacity: " +  
                                           initialCapacity); 

        // HashMap的最大容量只能是MAXIMUM_CAPACITY，哪怕传入的 > 最大容量
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;

        // 加载因子必须大于0,且不能为非数字  
        if (loadFactor <= 0 || Float.isNaN(loadFactor))  
            throw new IllegalArgumentException("Illegal load factor: " +  
                                           loadFactor);  
        // 设置 加载因子
        this.loadFactor = loadFactor;

        // 设置 扩容阈值
        // 注：此处不是真正的阈值，仅仅只是将传入的容量大小转化为：>传入容量大小的最小的2的幂，该阈值后面会重新计算
        // 下面会详细讲解 ->> 分析1
        this.threshold = tableSizeFor(initialCapacity); 

    }

    /**
     * 构造函数4：包含“子Map”的构造函数
     * 即 构造出来的HashMap包含传入Map的映射关系
     * 加载因子 & 容量 = 默认
     */

    public HashMap(Map<? extends K, ? extends V> m) {

        // 设置容量大小 & 加载因子 = 默认
        this.loadFactor = DEFAULT_LOAD_FACTOR; 

        // 将传入的子Map中的全部元素逐个添加到HashMap中
        putMapEntries(m, false); 
    }
}

   /**
     * 分析1：tableSizeFor(initialCapacity)
     * 作用：将传入的容量大小转化为：>传入容量大小的最小的2的幂
     * 与JDK 1.7对比：类似于JDK 1.7 中 inflateTable()里的 roundUpToPowerOf2(toSize)
     */
    static final int tableSizeFor(int cap) {
     int n = cap - 1;
     n |= n >>> 1;    //>>>操作符表示无符号右移，高位取0
     n |= n >>> 2;
     n |= n >>> 4;
     n |= n >>> 8;
     n |= n >>> 16;
     return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
}
```

上面的tableSizeFor()函数代码有点不太好看，总结起来就是找到大于或等于cap的最小2的幂，我们可以看看tableSizeFor的图解：

![选区_286.png](https://i.loli.net/2018/08/06/5b6724e21be79.png)

上面是 tableSizeFor 方法的计算过程图，这里`cap = 536,870,913 = 2^29 + 1`，多次计算后，算出`n + 1 = 1,073,741,824 = 2^30。通过图解应该可以比较容易理解这个方法的用途，这里就不多说了。

注：（同JDK 1.7类似） 

1. 此处仅用于接收初始容量大小（`capacity`）、加载因子(`Load factor`)，但仍无真正初始化哈希表，即初始化存储数组`table`
2. 此处先给出结论：**真正初始化哈希表（初始化存储数组table）是在第1次添加键值对时，即第1次调用put（）时。下面会详细说明**

### get操作

```java
   public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;

      // table已经初始化，长度大于0，根据hash寻找table中的项也不为空
     // index=(n-1)&hash  ==>求到数组table中的索引
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {

        // a. 看数组中头节点指向的元素是不是待查找的元素，是的话直接返回
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;

        // b. 如果first是TreeNode类型，则调用红黑树查找方法
        if ((e = first.next) != null) {
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);

            // c. 若红黑树中也没有，则通过遍历，到链表中寻找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

查找的核心逻辑是封装在 getNode 方法中，我们先来看看查找过程的第一步 - 确定桶位置，其实现代码如下：

```java
// index = (n - 1) & hash
first = tab[(n - 1) & hash]
```

这里通过`(n - 1)& hash`即可算出桶的在桶数组中的位置。HashMap 中桶数组的大小 length 总是2的幂，此时，`(n - 1) & hash` 等价于对 length 取余。但取余的计算效率没有位运算高，所以`(n - 1) & hash`也是一个小的优化。

在上面源码中，除了查找相关逻辑，还有一个计算 hash 的方法。这个方法源码如下：

```java
/**
 * 计算键的 hash 值
 */
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

