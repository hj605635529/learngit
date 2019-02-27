#   jvm学习

-  CMS GC时出现promotion failed跟concurrent mode failure

> CMS GC时出现promotion failed和concurrent mode failure

> 对于采用CMS进行旧生代GC的程序而言，尤其要注意GC日志中是否有promotion failed和concurrent mode failure两种状况，当这两种状况出现时可能会触发Full GC。

> promotion failed是在进行Minor GC时，survivor space放不下、对象只能放入旧生代，而此时旧生代也放不下造成的；concurrent mode failure是在执行CMS GC的过程中同时有对象要放入旧生代，而此时旧生代空间不足造成的。
>
> concurrent mode failure影响
>
> 老年代的垃圾收集器从CMS退化为Serial Old，所有应用线程被暂停，停顿时间变长。
>
> 可能原因及方案
> 原因1：CMS触发太晚
>
> 1. 方案：将-XX:CMSInitiatingOccupancyFraction=N调小；
>
> 原因2：空间碎片太多
> 方案：开启空间碎片整理，并将空间碎片整理周期设置在合理范围；
>
> 1. -XX:+UseCMSCompactAtFullCollection （空间碎片整理）
> 2. -XX:CMSFullGCsBeforeCompaction=n
>
> 原因3：垃圾产生速度超过清理速度
>
> 1. 晋升阈值过小；
> 2. Survivor空间过小，导致溢出；
> 3. Eden区过小，导致晋升速率提高；
> 4. 存在大对象；



![选区_754.png](https://i.loli.net/2019/02/24/5c72bebc4cbfd.png)



### hashmap面试题

1. 讲下HashMap中的put方法过程？

> - table为null的时候调用resize()方法进行初始化  数组大小为12
> - 对key求hash值， 然后计算数组下标
> - 如果没有碰撞，直接放到桶中
> - 如果碰撞了， 以链表的方式链接到后面
> - 如果链表的长度超过阀值 8 ，就把链表转换成红黑树。
> - 如果节点存在就替换旧值
> - 如果桶达到扩容的阀值， 就需要resize。

2. 扩容后，节点重hash为什么只可能分布在原索引位置与原索引位置+oldCap位置？

![选区_756.png](https://i.loli.net/2019/02/26/5c741a37ed9d3.png)

3. 为什么容量必须是2的n次冪？

   > 因为length为2的指数倍，所以`length-1`所对应的二进制位都为1，然后在与`hashCode(key)`做与运算，即可得到`[0,length)`内的索引,也就达到求模运算一样的结果。但是位运算效率更高。
   >
   > 但是直接用hashCode有一个问题， 比如table表的初始容量为16, 两个节点分别为0xABBA0000和0xBABA00， 与15相与的结果一样， 这是因为只用了低位做运算， 高位舍弃了，所以hashmap中用了hash函数，hash函数就是将高16位和低16位做一个异或运算， 然后得到一个结果，作用是：尽量让node落点分布均匀， 减少碰撞的一个概率， 如果碰撞概率高了， 导致数组下标下的链表长度太长。 

4. 讲下扩容过程？
> - 遍历所有的老桶， 如果某一个桶中只有一个元素，则将这个节点直接放入到新桶中
> - 如果这个桶下有多个元素， 遍历这个链表，链表下的元素只能放在两个位置， 一个是原下标的位置，另一个下标为<原下标+原容量>的位置。

5. 为啥hashMap是线程不安全的？

> jdk 1.7中hashMap插入元素是头插法，假设a,b两个线程都需要put操作， 并且put的元素的下标位置相同， 如果a线程刚标记要插入的元素的下一个位置， 还未插入元素就被切出去了， 然后b线程顺利的插入元素， 此时切回到a线程，先前标记的下一个位置此时已经是链表中第二个位置了， 然后再插入a线程元素， 把b线程插入的元素覆盖。jdk 1.8 是用的尾插法，没有这个问题。
>
> **死循环问题**

> jdk 1.7 中 我们假设有两个线程同时需要执行resize操作，我们原来的桶数量为2，记录数量为3，需要resize桶到4，原来的记录分别为：[3,A],[7,B],[5,C]，在原来的map里面，我们发现这三个entry都落到了第二个桶里面。

> 假设线程thread1执行到了transfer方法的Entry next = e.next这一句，然后时间片用完了，此时的e = [3,A], next = [7,B]。

> 线程thread2被调度执行并且顺利完成了resize操作，需要注意的是，此时的[7,B]的next为[3,A]。此时线程thread1重新被调度运行，此时的thread1持有的引用是已经被thread2 resize之后的结果。线程thread1首先将[3,A]迁移到新的数组上，然后再处理[7,B]，而[7,B]被链接到了[3,A]的后面，处理完[7,B]之后，就需要处理[7,B]的next了啊，而通过thread2的resize之后，[7,B]的next变为了[3,A]，此时，[3,A]和[7,B]形成了环形链表，在get的时候，如果get的key的桶索引和[3,A]和[7,B]一样，那么就会陷入死循环  jdk用的是尾插法， 没这个问题。

![选区_283.png](https://i.loli.net/2018/08/05/5b66f462c9926.png)

6. HashMap和Hashtable的区别：

> - HashMap允许key和value为null，Hashtable不允许。
> - HashMap的默认初始容量为16，Hashtable为11。
> - HashMap的扩容为原来的2倍，Hashtable的扩容为原来的2倍加1。
> - HashMap是非线程安全的，Hashtable是线程安全的。
> - HashMap的hash值重新计算过，Hashtable直接使用hashCode。
> - HashMap去掉了Hashtable中的contains方法。
> - HashMap继承自AbstractMap类，Hashtable继承自Dictionary类。



### 并发   

![1551187282531](/tmp/1551187282531.png)

![1551187649633](/tmp/1551187649633.png)



execute  submit方法的区别？

- 1. 返回值
  2.   执行的任务execute是自己程序员写的，但是submit执行的任务是futertast，对原来的任务的包装。





并发队列：  非阻塞队列， 当队列中满了的时候， 放入数据， 数据丢失， 如果队列中没有元素， 取数据， 得到的是null

​		      阻塞队列，当队列中满了的时候， 进行等待， 什么时候队列中有出队的数据， 然后把数据放入。 如果队列中没有数据， 等待，什么时候有数据， 再取出来。  

![选区_766.png](https://i.loli.net/2019/02/28/5c76d3c71d343.png)

线程池分类：

定时   