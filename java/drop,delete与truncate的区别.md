# drop,delete与truncate的区别

[TOC]

## 前提

今天晚上，产品让传一批数据到线上数据库上，原来的数据已经没用了， 我原本想把失效的数据删除，在网上查了下truncate比delete速度更快， 就使用了truncate,幸好被老大发现，没有造成实质的影响。因此总结下drop,delete,truncate的区别



-  delete语句执行删除的过程是每次从表中删除一行，并且同时将该行的删除操作作为事务记录在日志中保存以便进行进行回滚操作。truncate table 则一次性地从表中删除所有的数据并不把单独的删除操作记录记入日志保存，删除行是不能恢复的。并且在删除的过程中不会激活与表有关的删除触发器。执行速度快。

- 表和索引所占空间。当表被truncate后，这个表和索引所占用的空间会恢复到初始大小，而delete操作不会减少表或索引所占用的空间。drop语句将表所占用的空间全释放掉。

- drop直接删掉表 ;  truncate删除表中数据，再插入时自增长id又从1开始了;  delete删除表中数据，可以加where字句。

- 一般而言，drop > truncate > delete

- truncate 和delete只删除数据，而drop则删除整个表（结构和数据）。

- 在没有备份情况下，谨慎使用 drop 与 truncate。要删除部分数据行采用delete且注意结合where来约束影响范围。回滚段要足够大。要删除表用drop;若想保留表而将表中数据删除，如果于事务无关，用truncate即可实现。如果和事务有关，或想触发trigger,还是用delete。

-  truncate table +表名 速度快,而且效率高,因为: truncate table 在功能上与不带where 子句的delete语句相同：二者均删除表中的全部行。但 truncate table 比delete 速度快，且使用的系统和事务日志资源少。delete语句每次删除一行，并在事务日志中为所删除的每行记录一项。truncate table 通过释放存储表数据所用的数据页来删除数据，并且只在事务日志中记录页的释放。

- 对于由 foreign key 约束引用的表，不能使用 truncate table ，而应使用不带 where 子句的 delete 语句。由于truncate table 不记录在日志中，所以它不能激活触发器。

- drop table if exists xxx_book   如果数据库中存在xxx_book表，就把它从数据库中drop掉。备份sql中一般都有这样的语句，如果是数据库中有这个表，先drop掉，然后create表，然后再进行数据插入。

   

