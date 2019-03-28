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

## 5.主从复制

### 5.1主从复制的作用或好处

1、做数据的热备，作为后备数据库，主数据库服务器故障后，可切换到从数据库继续工作，避免数据丢失。
2、架构的扩展。业务量越来越大，I/O访问频率过高，单机无法满足，此时做多库的存储，降低磁盘I/O访问的频率，提高单个机器的I/O性能。
3、读写分离，使数据库能支撑更大的并发。在报表中尤其重要。由于部分报表sql语句非常的慢，导致锁表，影响前台服务。如果前台使用master，报表使用slave，那么报表sql将不会造成前台锁，保证了前台速度。

### 5.2 主从复制的原理：
1.数据库有个bin-log二进制文件，记录了所有sql语句。
2.我们的目标就是把主数据库的bin-log文件的sql语句复制过来。
3.让其在从数据的relay-log重做日志文件中再执行一次这些sql语句即可。
4.下面的主从配置就是围绕这个原理配置
5.具体需要三个线程来操作：
1.binlog输出线程:每当有从库连接到主库的时候，主库都会创建一个线程然后发送binlog内容到从库。在从库里，当复制开始的时候，从库就会创建两个线程进行处理：
2.从库I/O线程:当START SLAVE语句在从库开始执行之后，从库创建一个I/O线程，该线程连接到主库并请求主库发送binlog里面的更新记录到从库上。从库I/O线程读取主库的binlog输出线程发送的更新并拷贝这些更新到本地文件，其中包括relay log文件。

3.从库的SQL线程:从库创建一个SQL线程，这个线程读取从库I/O线程写到relay log的更新事件并执行。

可以知道，对于每一个主从复制的连接，都有三个线程。拥有多个从库的主库为每一个连接到主库的从库创建一个binlog输出线程，每一个从库都有它自己的I/O线程和SQL线程。
主从复制如图：

![image-20190329011651794](https://ws1.sinaimg.cn/large/006tKfTcly1g1j0izigqij30rh0al422.jpg)

原理图2,帮助理解! 

![image-20190329011713525](https://ws4.sinaimg.cn/large/006tKfTcly1g1j0jbamyoj30ru09ugoe.jpg)

步骤一：主库db的更新事件(update、insert、delete)被写到binlog
步骤二：从库发起连接，连接到主库
步骤三：此时主库创建一个binlog dump thread线程，把binlog的内容发送到从库
步骤四：从库启动之后，创建一个I/O线程，读取主库传过来的binlog内容并写入到relay log.
步骤五：还会创建一个SQL线程，从relay log里面读取内容，从Exec_Master_Log_Pos位置开始执行读取到的更新事件，将更新内容写入到slave的db



### 5.3主从复制的问题：

#### 5.3.1 主从同步延迟

表现：mysql做主从复制，但是发现主库修改了某些字段属性值，从从库查询，却拿到旧的值

原因：1.sql执行时间太长， 如：一个sql需要执行20s, 主库写完后， 从库需要花20s的时间才能更新完成， 这就导致20s的延迟。

​	   2.当主库的TPS并发较高时，产生的DDL数量超过slave一个sql线程所能承受的范围，那么延时就产生了，因为主库是可以并发执行的， 从库sql线程只有一个。

​	  3.网络延迟

解决方案：1.sql做优化

​		   2.从库使用更好的硬件设备，或增加slave

​		  3.更好的网络了。



#### 5.3.2 主从服务主服务器挂了怎么办？

3.Mysql5.5支持半同步

一个插件

一主多从主库宕机如何恢复，通过master.info确定新主库。

半同步下的一主多从恢复直接对设置半同步的从库确定为主库。
让某一个稳定从库和主库完全一致，即主库和这个从库都更新数据完毕，在返回给用户更新成功。

优点：

确保至少一个从库和主库数据一致

缺点：

主从之间网络延迟或者从库有问题时候，用户体验很差当然可以设置超时时间10秒

---------------------





