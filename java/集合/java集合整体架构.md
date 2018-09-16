# java集合整体架构

[TOC]

## 1.前言

Java中的集合包含多种数据结构，如链表、队列、哈希表等。从类的继承结构来说，可以分为两大类：

- 一类是继承自Collection接口，这类集合包含List、Set和Queue等集合类。
- 另一类是继承自Map接口，这主要包含了哈希表相关的集合类。

## 2.类图

## 2.1 List,Set和Queue类图



![集合.jpg](https://i.loli.net/2018/09/16/5b9e3aff068ce.jpg)

 **图中的绿色的虚线代表实现，绿色实线代表接口之间的继承，蓝色实线代表类之间的继承。**

 (1)List：我们用的比较多List包括ArrayList和LinkedList，这两者的区别也很明显，从其名称上就可以看出。ArrayList的底层的通过数组实现，所以其随机访问的速度比较快，但是对于需要频繁的增删的情况，效率就比较低了。而对于LinkedList，底层通过链表来实现，所以增删操作比较容易完成，但是对于随机访问的效率比较低。

(2)Queue：一般可以直接使用LinkedList完成，从上述类图也可以看出，LinkedList实现Deque，所以LinkedList具有双端队列的功能。PriorityQueue的特点是为每个元素提供一个优先级，优先级高的元素会优先出队列。

(3)Set：Set与List的主要区别是Set是不允许元素重复的，而List则可以允许元素重复的。判断元素的重复需要根据对象的hash方法和equals方法来决定。这也是我们通常要为集合中的元素类重写hashCode方法和equals方法的原因。HashSet和LinkedHashSet的区别在于后者可以保证元素插入集合的元素顺序与输出顺序保持一致。而TresSet的区别在于其排序是按照Comparator来进行排序的，默认情况下按照字符的自然顺序进行升序排列。

(4)Iterable：从这个图里面可以看到Collection类继承自Iterable，该接口的作用是提供元素遍历的功能，也就是说所有的集合类（除Map相关的类）都提供元素遍历的功能。Iterable里面包含了Iterator的迭代器，其源码如下：

## 2.2 Map类图

![map集合.jpg](https://i.loli.net/2018/09/16/5b9e3c96eae38.jpg)

Map类型的集合最大的优点在于其查找效率比较高，理想情况下可以实现O(1)的时间复杂度。Map中最常用的是HashMap，LinkedHashMap与HashMap的区别在于前者能够保证插入集合的元素顺序与输出顺序一致。这两者与TreeMap的区别在于TreeMap是根据键值进行排序的