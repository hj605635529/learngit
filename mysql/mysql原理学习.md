# mysql学习

[TOC]
## 1.sql解析顺序

https://www.cnblogs.com/annsshadow/p/5037667.html

## 2.数据库索引

- 索引的创建,查看，删除

>  alter table tbl_name  add index [indexName]  (columnname(length));

- 索引的查看

> show index from table;

- 索引的删除

> drop index index_name on table; 



![ 选区_780.png](https://i.loli.net/2019/03/03/5c7bed77bcb2a.png)

> - 哈希表虽然查找的速度很快，但是不适合范围查找。
> - 平衡二叉树的高度要比B树要高，因此需要更多次的磁盘io, 性能没有b树好。
> - b+树中叶子节点中有指针相连，因此范围查找的速度要比b树快。

![选区_768.png](https://i.loli.net/2019/03/01/5c7821b0818bf.png)

![选区_769.png](https://i.loli.net/2019/03/01/5c78222197c75.png)

![选区_770.png](https://i.loli.net/2019/03/01/5c78224fa234f.png)

![选区_767.png](https://i.loli.net/2019/03/01/5c78215cef1a5.png)

 

 ### 2.1 Innode的索引





![image-20190301112100483](https://ws4.sinaimg.cn/large/006tKfTcly1g0n4mwz5z3j319f0u0kam.jpg)

![image-20190301150930747](https://ws2.sinaimg.cn/large/006tKfTcly1g0nb8po6baj31u00u0h0t.jpg)

> **我们能看到辅助索引最终查找的结果就是主键键值， 然后在主键索引中从头开始找到要查找的记录。并不是图中那样直接从辅助索引一条线到要查找的记录**

### 2.2 MyISAM的索引

![选区_781.png](https://i.loli.net/2019/03/03/5c7bf57aa050e.png)



![选区_782.png](https://i.loli.net/2019/03/03/5c7bf628b556e.png)

![选区_783.png](https://i.loli.net/2019/03/03/5c7bf6d89b6dd.png)



![image-20190301111823271](https://ws3.sinaimg.cn/large/006tKfTcly1g0n4k8fb9pj31m60u07o9.jpg)

 #### 2.2.1Innodb和myisam区别？

1.innodb索引叶子节点存放数据， myisam叶子节点存放记录的位置，所以myisam会被innodb多一次磁盘io操作

2.myisam索引会存下表中数据的行数， innodb不会。

3.myisam索引支持的锁粒度为表锁， innodb锁粒度为行数。

4.myisam索引不支持事物， innodb支持事物。



### 2.3 那些字段适合做为索引

![选区_771.png](https://i.loli.net/2019/03/04/5c7bfc7dd5fc6.png)

### 2.4 索引失效

![选区_772.png](https://i.loli.net/2019/03/03/5c7b91b91e0c2.png)

- 关于第二条：

  ![选区_773.png](https://i.loli.net/2019/03/03/5c7b94e2d7cf2.png)

- 关于第四条

![选区_774.png](https://i.loli.net/2019/03/03/5c7b964b75097.png) 

- 关于第八条

![选区_775.png](https://i.loli.net/2019/03/03/5c7b98a0d1fba.png)

- 关于第九条   字符串不加单引号索引失效

> mysql自动会做类型转换，然后就回到第二条上了，在索引列上做操作导致索引失效。

- 那么在什么情况下能使用到覆盖索引呢？

>  where 条件 满足索引条件，必须所有条件都满足，select 查询的列必须也在索引上。 覆盖索引既索引上覆盖了select字段数据，查询时只要读取索引而无需访问数据行即可获得所有数据的索引

### 2.5 事务的隔离级别

![选区_784.png](https://i.loli.net/2019/03/04/5c7c04ce4985d.png)

脏读：

事务A读取到了事务B已修改但尚未提交的数据， 还在这个数据基础上做了操作， 此时，如果B事务回滚，A读取的数据无效，不符合一致性要求  

不可重复读

一个事务读取某些数据后的某个时间，再次读取原来读过的数据， 发现数据有改变， 这种现象叫做不可重复读，一句话：事务A读取到了事务B已经提交的修改数据， 不符合隔离性 

幻读

一个事务按相同的查询条件去查原来查过的数据， 发现有有其他事务插入满足条件的数据，这种现象就叫幻读。

一句话：事务A读取到了事务B已经新增的数据，不符合隔离性

RC实现原理是啥：每次执行读的时候都会创建一个视图， 视图中存放当前活跃的事物， 然后从undo.log中读取最新的版本号， 当该行事物id小于视图中最小id,输出。

<https://blog.csdn.net/joy0921/article/details/80128857>

### 2.6 explain性能分析

![选区_779.png](https://i.loli.net/2019/03/03/5c7bd7660ebc8.png)



![选区_785.png](https://i.loli.net/2019/03/04/5c7c076d30e64.png)



![选区_786.png](https://i.loli.net/2019/03/04/5c7c0814c1bdb.png)

![选区_787.png](https://i.loli.net/2019/03/04/5c7c09c814c41.png)

![选区_788.png](https://i.loli.net/2019/03/04/5c7c0a00dcaa4.png)

![选区_789.png](https://i.loli.net/2019/03/04/5c7c0a4e109a0.png)

> **总结下:我们一定要避免存在Using filesort，Using temporary这两种情况的存在 **



### 2.7 join、order by 、group by优化 

- join

![选区_790.png](https://i.loli.net/2019/03/04/5c7c0b34757d7.png)

>  show variables like 'join_%';

![选区_791.png](https://i.loli.net/2019/03/04/5c7c0b829ed21.png)

从图中我们能分析， 它查找b表是以遍历的形式去查找的，所有b的user_id需要加上索引，加速查找，当出现多个表的时候， 会出现join buffer缓成前面两个表生成的数据。

- order by

![选区_792.png](https://i.loli.net/2019/03/04/5c7c0c31676e1.png)

![选区_793.png](https://i.loli.net/2019/03/04/5c7c0cb725782.png)



- group by的前提是排序

- limit 

select * from user limit 10000, 10  这个做法的实现是全表扫描，取出10010行数据， 然后丢掉前面10000行数据。可以用where去优化：SELECT * FROM user where id> 10000 limit 10;



## 3. 数据结构软件：

https://www.cs.usfca.edu/~galles/visualization/Algorithms.html

## 4.案例分析


·[MySQL SQL优化之覆盖索引]   https://my.oschina.net/loujinhe/blog/1528233

>>>>>>> a2000ef342ffaf4eff1e5685aa36e8c8ce73fae7



以InnoDB来说，每个InnoDB表具有一个特殊的索引称为聚集索引。如果您的表上定义有主键，该主键索引是聚集索引。如果你不定义为您的表的主键时，MySQL取第一个唯一索引（unique）而且只含非空列（NOT NULL）作为主键，InnoDB使用它作为聚集索引。如果没有这样的列，InnoDB就自己产生一个这样的ID值，它有六个字节，而且是隐藏的，使其作为聚簇索引。

