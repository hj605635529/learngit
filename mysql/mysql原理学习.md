# mysql原理学习

[TOC]

![image-20190301112152643](https://ws3.sinaimg.cn/large/006tKfTcly1g0n4ntrc4zj31em0u0aob.jpg)



![image-20190301112100483](https://ws4.sinaimg.cn/large/006tKfTcly1g0n4mwz5z3j319f0u0kam.jpg)

![image-20190301150930747](https://ws2.sinaimg.cn/large/006tKfTcly1g0nb8po6baj31u00u0h0t.jpg)

> **我们能看到辅助索引最终查找的结果就是主键键值， 然后在主键索引中找到要查找的记录。**

主键索引和唯一索引的区别是啥？

![image-20190301111823271](https://ws3.sinaimg.cn/large/006tKfTcly1g0n4k8fb9pj31m60u07o9.jpg)

![image-20190301152532176](https://ws4.sinaimg.cn/large/006tKfTcly1g0nbpcukowj30xw0cgjxj.jpg)

mysql中每个表都有一个聚簇索引（clustered index ），除此之外的表上的每个非聚簇索引都是二级索引，又叫辅助索引（secondary indexes）。

以InnoDB来说，每个InnoDB表具有一个特殊的索引称为聚集索引。如果您的表上定义有主键，该主键索引是聚集索引。如果你不定义为您的表的主键时，MySQL取第一个唯一索引（unique）而且只含非空列（NOT NULL）作为主键，InnoDB使用它作为聚集索引。如果没有这样的列，InnoDB就自己产生一个这样的ID值，它有六个字节，而且是隐藏的，使其作为聚簇索引。