# LinkedList(1.8)源码剖析

[TOC]

## 0.前言

今天来介绍下LinkedList，LinkedList与ArrayList一样实现List接口，只是ArrayList是List接口的大小可变数组的实现，LinkedList是List接口链表的实现。基于链表实现的方式使得LinkedList在插入和删除时更优于ArrayList，而随机访问则比ArrayList逊色些。
 构造图如下:
 蓝色线条：继承
 绿色线条：接口实现

 ![LinkedList.png](https://i.loli.net/2018/09/13/5b9947da5e317.png)

  

## 1.LinkedList定义

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
```

- LinkedList 是一个继承于AbstractSequentialList的**双向链表**。（在1.6中是双向循环链表， 1.7之后就变成了双向链表了）它也可以被当作堆栈、队列或双端队列进行操作。
- LinkedList 实现 List 接口，能对它进行队列操作。
- LinkedList 实现 Deque 接口，即能将LinkedList当作双端队列使用。
- LinkedList 实现了Cloneable接口，即覆盖了函数clone()，能克隆。
- LinkedList 实现java.io.Serializable接口，这意味着LinkedList支持序列化，能通过序列化去传输。
- LinkedList 是非同步的

 ## 2.LinkedList属性

### 2.1LinkedList类的内部类

```java
private static class Node<E> {
        E item; // 数据域
        Node<E> next; // 后继
        Node<E> prev; // 前驱
        
        // 构造函数，赋值前驱后继
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
```

内部类Node即是实际结点，用于存放实际数据

### 2.2LinkedList类的属性

```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable
{
    // 实际元素个数
    transient int size = 0;
    // 头结点
    transient Node<E> first;
    // 尾结点
    transient Node<E> last;
}
```

说明：LinkedList的属性不多，其中实际元素个数、头结点、尾结点均用transient关键字修饰，即意味着在序列化的时候该域不会被序列化

## 3.构造函数

```java
//无参构造函数
public LinkedList() {
}

//LinkedList(Collection<?extends E>)型构造函数
public LinkedList(Collection<? extends E> c) {
        // 调用无参构造函数
        this();
        // 添加集合中所有的元素
        addAll(c);
    }
```



## LinkedList核心方法分析

 ```java
 public boolean add(E e) {
        // 添加到末尾
        linkLast(e);
        return true;
    }
 ```

add()方法用于向LinkedList中添加一个元素，且添加在尾部，具体添加通过linkedLast()方法实现，其源码如下：

```java
void linkLast(E e) {
        // 保存尾结点，l为final类型，不可更改
        final Node<E> l = last;
        // 新生成结点的前驱为l,后继为null
        final Node<E> newNode = new Node<>(l, e, null);
        // 重新赋值尾结点
        last = newNode;    
        if (l == null) // 尾结点为空
            first = newNode; // 赋值头结点
        else // 尾结点不为空
            l.next = newNode; // 尾结点的后继为新生成的结点
        // 大小加1    
        size++;
        // 结构性修改加1
        modCount++;
    }
```

addAll()方法：

addAll()方法有两个重载函数：

1.`public boolean addAll(Collection<? extends E> c)`

2.`public boolean addAll(int index, Collection<? extends E> c) `

我们平常调用的1方法会转换为2方法。

```java
public boolean addAll(Collection<? extends E> c) {
    return addAll(size, c);
}
```

 

```java
 public boolean addAll(int index, Collection<? extends E> c) {
     //检查插入的位置是否合法  
     checkPositionIndex(index);
		//将集合转换成数组
        Object[] a = c.toArray();
     	//保存集合大小
        int numNew = a.length;
        if (numNew == 0)   //// 集合为空，直接返回
            return false;

        Node<E> pred, succ;// 前驱，后继
        if (index == size) {  // 如果插入位置为链表末尾，则后继为null，前驱为尾结点
            succ = null;
            pred = last;
        } else {            // 插入位置为其他某个位置
            succ = node(index);   // 寻找到该位置的节点
            pred = succ.prev;   // 保存该结点的前驱
        }

        for (Object o : a) {
            @SuppressWarnings("unchecked") E e = (E) o;  // 向下转型
             // 生成新结点
            Node<E> newNode = new Node<>(pred, e, null);
            // 表示在第一个元素之前插入(索引为0的结点)
            if (pred == null)
                first = newNode;
            else
                pred.next = newNode;
            pred = newNode;
        }

        if (succ == null) {    //// 表示在最后一个元素之后插入
            last = pred;
        } else {
            pred.next = succ;
            succ.prev = pred;
        }

        size += numNew;
        modCount++;
        return true;
    }
```

参数中的index表示在索引下标为index的结点的前面插入，在addAll()方法中，addAll()方法还会调用node()方法,此方法是根据索引下标找到该结点后再返回，具体源码如下：

```java
Node<E> node(int index) {
        // 判断插入的位置在链表前半段或者是后半段
        if (index < (size >> 1)) { // 插入位置在前半段
            Node<E> x = first; 
            for (int i = 0; i < index; i++) // 从头结点开始正向遍历
                x = x.next;
            return x; // 返回该结点
        } else { // 插入位置在后半段
            Node<E> x = last; 
            for (int i = size - 1; i > index; i--) // 从尾结点开始反向遍历
                x = x.prev;
            return x; // 返回该结点
        }
    }
```

说明：在根据索引查找结点时，会有一个小优化，结点在前半段则从头开始遍历，在后半段则从尾开始遍历，这样就保证了只需要遍历最多一半结点就可以找到指定索引的结点。



remove方法：

 ```java
public boolean remove(Object o) {
        if (o == null) {
            for (Node<E> x = first; x != null; x = x.next) {
                if (x.item == null) {
                    unlink(x);
                    return true;
                }
            }
        } else {
            for (Node<E> x = first; x != null; x = x.next) {
                if (o.equals(x.item)) {
                    unlink(x);
                    return true;
                }
            }
        }
        return false;
    }
 ```

在调用remove移除结点时，会调用unlink()方法，将指定结点从链表中断开，其具体实现如下：

```java
E unlink(Node<E> x) {
        // 保存结点的元素
        final E element = x.item;
        // 保存x的后继
        final Node<E> next = x.next;
        // 保存x的前驱
        final Node<E> prev = x.prev;
        
        if (prev == null) { // 前驱为空，表示删除的结点为头结点
            first = next; // 重新赋值头结点
        } else { // 删除的结点不为头结点
            prev.next = next; // 赋值前驱结点的后继
            x.prev = null; // 结点的前驱为空，切断结点的前驱指针
        }

        if (next == null) { // 后继为空，表示删除的结点为尾结点
            last = prev; // 重新赋值尾结点
        } else { // 删除的结点不为尾结点
            next.prev = prev; // 赋值后继结点的前驱
            x.next = null; // 结点的后继为空，切断结点的后继指针
        }

        x.item = null; // 结点元素赋值为空
        // 减少元素实际个数
        size--; 
        // 结构性修改加1
        modCount++;
        // 返回结点的旧元素
        return element;
    }
```



## 迭代操作

