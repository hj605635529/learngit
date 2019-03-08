# ConcurrentHashMap源码剖析

```java
/** 
   * 盛装Node元素的数组 它的大小是2的整数次幂 
   * Size is always a power of two. Accessed directly by iterators. 
   */  
  transient volatile Node<K,V>[] table;  
  
/** 
   hash表初始化或扩容时的一个控制位标识量。 
   负数代表正在进行初始化或扩容操作 
   -1代表正在初始化 
   -N 表示有N-1个线程正在进行扩容操作 当线程扩容时，会把这个sizeCtrl设置为负数。
   正数或0代表hash表还没有被初始化，这个数值表示初始化或者扩容阀值。 
    
   */  
  private transient volatile int sizeCtl;   
  // 以下两个是用来控制扩容的时候 单线程进入的变量  
   /** 
   * The number of bits used for generation stamp in sizeCtl. 
   * Must be at least 6 for 32bit arrays. 
   */  
  private static int RESIZE_STAMP_BITS = 16;  
/** 
   * The bit shift for recording size stamp in sizeCtl. 
   */  
  private static final int RESIZE_STAMP_SHIFT = 32 - RESIZE_STAMP_BITS;  
    
    
  /* 
   * Encodings for Node hash fields. See above for explanation. 
   */  
  static final int MOVED     = -1; // hash值是-1，表示这是一个forwardNode节点  
  static final int TREEBIN   = -2; // hash值是-2  表示这时一个TreeBin节点  
```



```java
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;
    final K key;
    volatile V val;                 //保证可见性
    volatile Node<K,V> next;

    Node(int hash, K key, V val, Node<K,V> next) {
        this.hash = hash;
        this.key = key;
        this.val = val;
        this.next = next;
    }
}    
```



## 初始化（当第一次put操作的时候才会对数组进行分配内存）

```java
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        if ((sc = sizeCtl) < 0)
            //其他线程在初始化， 当前线程放弃cpu,等待它完成初始化操作。
            Thread.yield(); // lost initialization race; just spin
        //利用cas保证初始化操作只能是一个线程完成， -1表示在初始化。
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    //用这种移位操作来算出阀值。
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```



## 添加元素

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        Node<K,V> f; int n, i, fh;
        if (tab == null || (n = tab.length) == 0)
            //1.初始化动作
            tab = initTable();
        //2.如果当前节点没有原始， 则插入新的节点
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            if (casTabAt(tab, i, null,
                         new Node<K,V>(hash, key, value, null)))
                break;                   // no lock when adding to empty bin
        }
        //3. 如果发现有其他的线程在扩容， 则协助扩容
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            //4.加锁插入元素，如果是链表，则遍历链表，如果有相同的key,则用新的value替换旧值，否则插入到链表的尾部，如果是树， 以树的形式插入新的节点。
            V oldVal = null;
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    if (fh >= 0) {
                        binCount = 1;
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            //5.如果链表的元素大于8，转换为树
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    //6.判断是否要扩容
    addCount(1L, binCount);
    return null;
}
```

## 判断是否扩容

```java
private final void addCount(long x, int check) {
    CounterCell[] as; long b, s;
    if ((as = counterCells) != null ||
        !U.compareAndSwapLong(this, BASECOUNT, b = baseCount, s = b + x)) {
        CounterCell a; long v; int m;
        boolean uncontended = true;
        if (as == null || (m = as.length - 1) < 0 ||
            (a = as[ThreadLocalRandom.getProbe() & m]) == null ||
            !(uncontended =
              U.compareAndSwapLong(a, CELLVALUE, v = a.value, v + x))) {
            fullAddCount(x, uncontended);
            return;
        }
        if (check <= 1)
            return;
        s = sumCount();
    }
    if (check >= 0) {
        Node<K,V>[] tab, nt; int n, sc;
        while (s >= (long)(sc = sizeCtl) && (tab = table) != null &&
               (n = tab.length) < MAXIMUM_CAPACITY) {
            int rs = resizeStamp(n);
            if (sc < 0) {
                if ((sc >>> RESIZE_STAMP_SHIFT) != rs || sc == rs + 1 ||
                    sc == rs + MAX_RESIZERS || (nt = nextTable) == null ||
                    transferIndex <= 0)
                    break;
                if (U.compareAndSwapInt(this, SIZECTL, sc, sc + 1))
                    //协助扩容
                    transfer(tab, nt);
            }
            //第一次扩容， nextTable还是空的。将sizeCtl由正数改为负数。
            else if (U.compareAndSwapInt(this, SIZECTL, sc,
                                         (rs << RESIZE_STAMP_SHIFT) + 2))
                transfer(tab, null);
            s = sumCount();
        }
    }
}
```



## 扩容

```java
private final void transfer(Node<K,V>[] tab, Node<K,V>[] nextTab) {
    int n = tab.length, stride;
      //根据cpu个数及MIN_TRANSFER_STRIDE值来计算stride，MIN_TRANSFER_STRIDE为16
       // stride的大小代表该线程所负责的table数组扩容范围
    if ((stride = (NCPU > 1) ? (n >>> 3) / NCPU : n) < MIN_TRANSFER_STRIDE)
        stride = MIN_TRANSFER_STRIDE; 
    
    //只有一个线程负责创建新的数组操作，这个在addCount中利用cas做判断了。
    if (nextTab == null) {            
        try {
            @SuppressWarnings("unchecked")
            Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n << 1];
            nextTab = nt;
        } catch (Throwable ex) {      // try to cope with OOME
            sizeCtl = Integer.MAX_VALUE;
            return;
        }
        nextTable = nextTab;
        transferIndex = n;
    }
    
    int nextn = nextTab.length;
    ForwardingNode<K,V> fwd = new ForwardingNode<K,V>(nextTab);
    boolean advance = true;    //并发扩容的标记， true表示这个节点已经处理完了。
    boolean finishing = false; //finishing表示扩容结束了
    
    for (int i = 0, bound = 0;;) {
        Node<K,V> f; int fh;
        
        //这个while循环主要用来为每个线程分配处理区域， 默认一个线程处理16个节点。
        while (advance) {
            int nextIndex, nextBound;
            //当前线程处理的区域还没结束， 继续处理
            if (--i >= bound || finishing)
                advance = false;
            //区域已经分配完了， 后面的线程i=-1,直接结束
            else if ((nextIndex = transferIndex) <= 0) {
                i = -1;
                advance = false;
            }
            //分配处理区域[0,15],[16,31]这样分配
            else if (U.compareAndSwapInt
                     (this, TRANSFERINDEX, nextIndex,
                      nextBound = (nextIndex > stride ?
                                   nextIndex - stride : 0))) {
                bound = nextBound;
                i = nextIndex - 1;
                advance = false;
            }
        }
        
        //这个if用来结束扩容操作的。刚开始数组大小为16，其实第二个线程进来扩容的时候， 已经没有区域给它处理了， i直接=-1， 
        if (i < 0 || i >= n || i + n >= nextn) {
            int sc;
            //最后一个线程 finishing为true.
            if (finishing) {
                nextTable = null;
                table = nextTab;
                //更新阀值了。
                sizeCtl = (n << 1) - (n >>> 1);
                return;
            }
            //线程处理完自己的区域时， sizeCtrl-1， 如果不是最后一个线程就直接退出。
            if (U.compareAndSwapInt(this, SIZECTL, sc = sizeCtl, sc - 1)) {
                //最后一个线程sc-2 ==  resizeStamp(n) << RESIZE_STAMP_SHIFT
                if ((sc - 2) != resizeStamp(n) << RESIZE_STAMP_SHIFT)
                    return;
                
                //最后一个线程把finishing标志置为true。
                finishing = advance = true;
                //重新检查一遍， 看是否原来数组所有的节点都已经移动到新的数组上了。
                i = n; // recheck before commit
            }
        }
        //这个节点是空的，给它打上标记
        else if ((f = tabAt(tab, i)) == null)
            advance = casTabAt(tab, i, null, fwd);
        //这个节点已经处理过了， 不需要再处理了， 主要用来第二次检查判断是否还有节点没有处理完。
        else if ((fh = f.hash) == MOVED)
            advance = true; // already processed
        else {
            //给节点加锁处理。
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    Node<K,V> ln, hn;
                    if (fh >= 0) {
                        int runBit = fh & n;
                        Node<K,V> lastRun = f;
                        for (Node<K,V> p = f.next; p != null; p = p.next) {
                            int b = p.hash & n;
                            if (b != runBit) {
                                runBit = b;
                                lastRun = p;
                            }
                        }
                        //runBit==0表示这个节点放在扩容后数组i位置
                        if (runBit == 0) {
                            ln = lastRun;
                            hn = null;
                        }
                        //否则表示这个节点放在扩容后数组i+n位置上。n表示原数组大小
                        else {
                            hn = lastRun;
                            ln = null;
                        }
                        //反序链表
                        for (Node<K,V> p = f; p != lastRun; p = p.next) {
                            int ph = p.hash; K pk = p.key; V pv = p.val;
                            if ((ph & n) == 0)
                                ln = new Node<K,V>(ph, pk, pv, ln);
                            else
                                hn = new Node<K,V>(ph, pk, pv, hn);
                        }
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                    else if (f instanceof TreeBin) {
                        TreeBin<K,V> t = (TreeBin<K,V>)f;
                        TreeNode<K,V> lo = null, loTail = null;
                        TreeNode<K,V> hi = null, hiTail = null;
                        int lc = 0, hc = 0;
                        for (Node<K,V> e = t.first; e != null; e = e.next) {
                            int h = e.hash;
                            TreeNode<K,V> p = new TreeNode<K,V>
                                (h, e.key, e.val, null, null);
                            if ((h & n) == 0) {
                                if ((p.prev = loTail) == null)
                                    lo = p;
                                else
                                    loTail.next = p;
                                loTail = p;
                                ++lc;
                            }
                            else {
                                if ((p.prev = hiTail) == null)
                                    hi = p;
                                else
                                    hiTail.next = p;
                                hiTail = p;
                                ++hc;
                            }
                        }
                        ln = (lc <= UNTREEIFY_THRESHOLD) ? untreeify(lo) :
                            (hc != 0) ? new TreeBin<K,V>(lo) : t;
                        hn = (hc <= UNTREEIFY_THRESHOLD) ? untreeify(hi) :
                            (lc != 0) ? new TreeBin<K,V>(hi) : t;
                        setTabAt(nextTab, i, ln);
                        setTabAt(nextTab, i + n, hn);
                        setTabAt(tab, i, fwd);
                        advance = true;
                    }
                }
            }
        }
    }
}
```

![image-20190308153019728](https://ws1.sinaimg.cn/large/006tKfTcly1g0vf6jbcrcj31k00qcgrr.jpg)



## 获取元素

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null)
    	{ //判断第一个节点的值是否为我们要查找的元素
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                return e.val;
        }
        //在树上查找
        else if (eh < 0)
            return (p = e.find(h, key)) != null ? p.val : null;
        //在链表查找
        while ((e = e.next) != null) {
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```



https://blog.csdn.net/sinat_34976604/article/details/80971620

https://blog.csdn.net/jianghuxiaojin/article/details/52006118