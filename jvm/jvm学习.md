# jvm学习

- Java虚拟机JVM之server模式与client模式的区别

  > JVM Server模式下应用启动慢但运行速度快，JVM Client模式下应用启动快但运行速度要慢些，推荐：服务器上请以Server模式运行，面客户端或GUI模式下就以Client模式运行
  >
  > -Server模式启动时，速度较慢，但是一旦运行起来后，性能将会有很大的提升，原因是：当虚拟机在-Client模式的时候，使用的是一个代号为C1的轻量级编译器，而-Server模式启动的虚拟机采用相对重量级代号为C2的编译器，C2比C1编译器编译的相对彻底，服务起来之后，性能高。
  >
  > https://www.jb51.net/article/129592.htm

- 多cpu和多核cpu有什么区别？

  > 双核心是在一个处理器里拥有两个处理器核心，核心是两个，但是其他硬件还都是两个核心在共同拥有，而双CPU则是真正意义上的双核心，不光是处理器核心是两个，其他例如缓存等硬件配置也都是双份的。
  >
  > https://blog.csdn.net/qingkongyeyue/article/details/73513060

- jvm常用命令有哪些？

  > Jinfo   查看参数信息
  >
  > Jstat -gcutil 12345  2000 10  每隔2秒查看12345进程的参数信息， 共查看10次。
  >
  > Jmap -heap  查看堆的大小情况。-dump  dump文件
  >
  > Jstack 生成java虚拟机当前时刻的线程快照

- 双亲委派机制

  > 当类加载器收到类加载的请求的时候， 它并不会去加载这个请求， 而是把这个请求交给父类去加载， 上层类加载器重复这个操作， 最终请求将到达顶层的启动类加载器， 只有父类加载器加载这个请求失败时，子类加载器才会去加载这个请求。

- 优势：
  >  类加载器有层级关系， 类加载请求一层层向上传递，避免子类加载器重复加载。
  >
  >  考虑安全因素， 当你在当前项目自定义java.lang.String类， 因为双亲委派机制， 启动类加载器去加载jre/bin/rt.jar包中的String, 类加载只加载一次， 不会加载你自己写的类， 防止核心库被随意篡改

- 沙箱机制

  >  沙箱机制是由基于双亲委派机制上 采取的一种JVM的自我保护机制,假设你要写一个java.lang.String 的类,由于双亲委派机制的原理,此请求会先交给Bootstrap试图进行加载,但是Bootstrap在加载类时首先通过包和类名查找rt.jar中有没有该类,有则优先加载rt.jar包中的类,因此就保证了java的运行机制不会被破坏.

- 对象是否可GC？

  > 引用计数法判断  不能解决循环引用的问题
  >
  > 可达性分析， 从gc Roots开始， 做可达性分析， 如果某个对象不在引用连中， 表示这个对象可以被gc

- 那些对象可以作为GCroot呢？

  > 虚拟机栈（栈帧中的本地变量表）中的引用的对象
  > 方法区中的类静态属性引用的对象
  > 方法区中的常量引用的对象
  > 本地方法栈中JNI（Native方法）的引用对象

- Minor GC 和Full GC?

  > 当在新生代的eden区为对象分配空间时，分配不下，会触发一次minor gc。 

  > 如果创建一个大对象，Eden区域当中放不下这个大对象，会直接保存在老年代当中，如果老年代空间也不足，就会触发Full GC
  >
  > 显示调用System.gc
  >
  >  出现promotion failure 和concurrent model failure， 接下去就会发生Full GC. 
  >
  > 如果有持久代空间的话，系统当中需要加载的类，调用的方法很多，同时持久代当中没有足够的空间，就出触发一次Full GC
  >

- 什么是promotion failure， concurrent model failure?

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

- 垃圾回收算法？

  > 复制算法 -- 空间利用率只有50%， 当一半空间已经分配不了内存时， 将空间内存活的对象复制到另一半空间中， 优势是：简单，不会产生内存碎片。
  >
  > 标记清除算法：遍历所有的GCRoots，将GCRoots可达的对象标记为存活的对象，然后在遍历堆中所有的对象，将没有标记对象清除。 标记和清除两个阶段都要消耗时间， 并且会产生内存碎片。
  >
  > 标记整理算法：遍历所有的GCRoots，将GCRoots可达的对象标记为存活的对象 然后将存活的对象移动到一端。直接清除端以为的对象。
  >
  > 分代收集算法： 比如新生代使用复制算法， 老年代使用标记清除算法。

- 垃圾收集器？

  > ![image-20190323192941274](https://ws4.sinaimg.cn/large/006tKfTcly1g1cye7i29yj30ng0exq54.jpg)

- 垃圾回收器G1？

  > G1建立可预测停顿时间模型， 可以在m毫秒的时间内进行n毫秒的垃圾回收，不需要和其他垃圾收集器配合使用， 在新生代和老年代都使用G1, 而且G1使用的标记整理算法， 不会产生内存碎片。有下面几个回收步骤： 1.初始标记， 2.并发标记  3.最终标记  4. 筛选回收

- CMS收集器的缺点？

  ![选区_754.png](https://i.loli.net/2019/02/24/5c72bebc4cbfd.png)

- jvm参数？

  > -Xms8192m   设置JVM初始堆内存为8G
  >
  > -Xmx8192m  设置JVM最大堆内存为8G
  >
  >  -Xmn1024m  设置年轻代大小为1G
  >
  > -XX:+UseConcMarkSweepGC  老年代使用cms
  >
  > -XX:+UseParNewGC   新生代使用parNew
  >
  > -XX:PermSize=300m  设置永久代为300M内存空间
  >
  >  -XX:ParallelGCThreads=20 设置并行GC时的线程数
  >
  > -XX:CMSInitiatingOccupancyFraction=75 老年代使用75%的使用触发full gc 
  >
  > -verbose:gc  输出虚拟机中GC的详情情况
  >
  >  -XX:+PrintGCDateStamps  带有时间搓
  >
  > -XX:+PrintGCDetails    输出详细情况
  >
  > -Xloggc:$CATALINA_BASE/logs/gc.log    日志输出地方
  >
  >  -XX:+DisableExplicitGC   不响应 System.gc() 代码
  >
  > 

![image-20190217135230700](https://ws4.sinaimg.cn/large/006tKfTcly1g09dkwhuncj30s8055759.jpg)

- Java 内存结构？

  > https://blog.csdn.net/sinat_33921105/article/details/82819435

- OOM 出现的有哪些场景？为什么会发生？

  > **每个方法执行都会创建一个栈帧，用于存放局部变量表，操作栈，动态链接，方法出口等**。**每个方法从被调用，直到被执行完。对应着一个栈帧在虚拟机中从入栈到出栈的过程**。
  >
  > 通常说的栈就是指局部变量表部分，存放编译期间可知的8种基本数据类型，及对象引用和指令地址。局部变量表是在编译期间完成分配，当进入一个方法时，这个栈中的局部变量分配内存大小是确定的。
  >
  > 会有两种异常StackOverFlowError和 OutOfMemoneyError。当线程请求栈深度大于虚拟机所允许的深度就会抛出StackOverFlowError错误；虚拟机栈动态扩展，当扩展无法申请到足够的内存空间时候，抛出OutOfMemoneyError。

- java 引用类型

  > 强引用：我们创建的对象就是强引用
  >
  > 软引用：当内存足够的时候，对象不会被回收， 内存不够的时候， 对象被回收
  >
  > 弱引用：只要进行垃圾回收，就会被回收
  >
  > 虚引用： 和没有引用差不多
  >
  > https://www.cnblogs.com/liyutian/p/9690974.html

- Minor GC、Major GC和Full GC之间的区别

  > http://www.importnew.com/15820.html
  >
  > https://www.zhihu.com/question/35164211

- 为什么新生代需要两个s区？

  > ![img](https://ws4.sinaimg.cn/large/006tNc79ly1fzqyzfyidaj30k008wagc.jpg)
  >
  > ![image-20190201160657369](/Users/huangjia/Library/Application Support/typora-user-images/image-20190201160657369.png)
  >
  > 为什么需要From 和 To 两个平行的区呢，为什么不直接从Survivor 移到 Old？ 这样设计的好处是什么？难道是因为在移动对象的时候需要压缩调整对象空间，所以这种整体移动的设计会快一点吗？希望大家一起来讨论一下 ^_^
  >
  > 之所以要分两个Survivor，而不是直接从Survivor直接移到old区域，原因是old区域内的对象都是经过若干次yong GC之后存活下来的对象，并不是每一次yong GC存活下来的对象都需要移动到old区域内，所以需要Survivor1和Survivor2来保证Yong内存中的复制算法的实行，提高清除效率。

- 担保机制？

  > 对象分配时，优先在eden区分配，当eden区空间不足时，触发一次minor gc。minor gc会标记eden区及from survivor区中不可以回收的对象，准备放入to survivor区。 
  >
  > 当eden和from survivor中存活对象，在to survivor中放不下时，需要利用担保机制，将这些对象直接放到老年代。然后再在eden上分配对象



