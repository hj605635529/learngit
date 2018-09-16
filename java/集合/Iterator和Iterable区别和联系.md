# Iterator和Iterable区别和联系

[TOC]

## 1.前言

 Iterator模式是用于遍历集合类的标准访问方法。它可以把**访问逻辑从不同类型的集合类中抽象出来，从而避免向客户端暴露集合的内部结构。**

 如果没有使用Iterator，遍历一个数组的方法是使用索引： for(int i=0; i<array.size(); i++) { ... get(i) ... }
​ 而访问一个链表（LinkedList）又必须使用while循环： while((e=e.next())!=null) { ... e.data() ... } 
 以上两种方法客户端都必须事先知道集合的内部结构，访问代码和集合本身是**紧耦合**，无法将访问逻辑从集合类	        和客户端代码中分离出来，每一种集合对应一种遍历方法，客户端代码无法复用。
 更恐怖的是，如果以后需要把ArrayList更换为LinkedList，则原来的客户端代码必须全部重写。

## 2.Iterator模式

  解决以上问题，Iterator模式总是用同一种逻辑来遍历集合： for(Iterator it = c.iterator(); it.hasNext(); ) { ... } 

  客户端从不直接和集合类打交道，它总是控制Iterator，向它发送"向前"，"向后"，"取当前元素"的命令，就可以间接遍历整个集合。

## 3.Iterator接口

```java
public interface Iterator<E> {
    /**
     * Returns {@code true} if the iteration has more elements.
     * (In other words, returns {@code true} if {@link #next} would
     * return an element rather than throwing an exception.)
     *
     * @return {@code true} if the iteration has more elements
     */
    boolean hasNext();

    /**
     * Returns the next element in the iteration.
     *
     * @return the next element in the iteration
     * @throws NoSuchElementException if the iteration has no more elements
     */
    E next();

    /**
     * Removes from the underlying collection the last element returned
     * by this iterator (optional operation).  This method can be called
     * only once per call to {@link #next}.  The behavior of an iterator
     * is unspecified if the underlying collection is modified while the
     * iteration is in progress in any way other than by calling this
     * method.
     *
     * @implSpec
     * The default implementation throws an instance of
     * {@link UnsupportedOperationException} and performs no other action.
     *
     * @throws UnsupportedOperationException if the {@code remove}
     *         operation is not supported by this iterator
     *
     * @throws IllegalStateException if the {@code next} method has not
     *         yet been called, or the {@code remove} method has already
     *         been called after the last call to the {@code next}
     *         method
     */
    default void remove() {
        throw new UnsupportedOperationException("remove");
    }
}
```

依赖前两个方法就能完成遍历，典型的代码如下:

    `for(Iterator it = c.iterator(); it.hasNext(); ) { Object o = it.next(); // 对o的操作... } `

 每一种集合类返回的Iterator具体类型可能不同，Array可能返回ArrayIterator，Set可能返回 SetIterator，Tree可能返回TreeIterator，但是它们都实现了Iterator接口，因此，客户端不关心到底是哪种 Iterator，它只需要获得这个Iterator接口即可，这就是面向对象的威力。 

所有集合类都实现了 Collection 接口，而 Collection 继承了 Iterable 接口。

## 3.Iterable接口

```java
public interface Iterable<T> {
    /**
     * Returns an iterator over elements of type {@code T}.
     *
     * @return an Iterator.
     */
    Iterator<T> iterator();
}
```

## 4.具体iterator实现

比如 ArrayList，则在内部维护了一个 Itr 内部类，该类实现 Iterator 接口，它的hasNext() 和 next() 方法是和 ArrayList 实现相耦合的。当调用 ArrayList 对象的 iterator() 方法的时候，返回该类 Itr 的一个实例，从而实现遍历 ArrayList 的功能。

```java
public Iterator<E> iterator() {
    return new Itr();
}

/**
 * An optimized version of AbstractList.Itr
 */
private class Itr implements Iterator<E> {
    int cursor;       // index of next element to return
    int lastRet = -1; // index of last element returned; -1 if no such
    int expectedModCount = modCount;

    Itr() {}

    public boolean hasNext() {
        return cursor != size;
    }

    @SuppressWarnings("unchecked")
    public E next() {
        checkForComodification();
        int i = cursor;
        if (i >= size)
            throw new NoSuchElementException();
        Object[] elementData = ArrayList.this.elementData;
        if (i >= elementData.length)
            throw new ConcurrentModificationException();
        cursor = i + 1;
        return (E) elementData[lastRet = i];
    }
```

## 5.总结

**为什么一定要去实现Iterable这个接口呢？为什么不直接实现Iterator接口呢？**

​        看一下JDK中的集合类，比如List一族或者Set一族，都是实现了Iterable接口。但并不直接实现Iterator接口。 实现了Iterable的接口，可以使用foreach()进行遍历，实际上编译器也是转换成iterator的hasNext()和next()的调用。

​	主要是为了迭代器之间不会相互干扰，next()方法依赖当前位置，那么实现类就需要当前的 位置指针，例如，当对某个list集合中，包含了10个元素。在一个  方法中，遍历前面若干元素。当我想要在另一个方法中，每次都重新开始遍历。这个时候就很难做到，除非每次都是重置reset 这个设计是非常不合理的。因此使用一个Iterable，每次都创建新的迭代器，之间不会相互干扰。

**`Iterable`只是返回了`Iterator`接口的一个实例，这里很是奇怪，为什么不把两个接口合二为一，直接在`Iterable`里面定义`hasNext()`,`next()`等方法呢？**

实现了`Iterable`的类可以在实现多个`Iterator`内部类，例如`LinkedList`中的`ListItr`和`DescendingIterator`两个内部类，就分别实现了双向遍历和逆序遍历。通过返回不同的`Iterator`实现不同的遍历方式，这样更加灵活。如果把两个接口合并，就没法返回不同的`Iterator`实现类了。

 

 

 

 

 