# IDEA 用法总结

[TOC]

## 1.远程debug注意

idea中设置需要连接的服务器主机ip,以及这个连接请求走的端口号。

![选区_419.png](https://i.loli.net/2018/09/06/5b8ffdf90a21b.png)

服务器上：

socat TCP4-LISTEN:9113,fork,range=100.80.181.199/32 TCP4:127.0.0.1:9005

这句话的意思表示监听9113端口， 将这个9113端口发送的请求给9005端口， 9005端口是服务器上tomcat的端口号，也就是将这个远程debug请求给tomcat处理， 最终实现远程debug。range表示接受那台机器上请求，也就是那台机器上发送的远程debug请求能被接受。socat实现一个端口转发的功能。

**注意点：当你不需要远程debug的时候，一定要注意将idea的远程debug关了，本人今天下班的时候，只把idea关闭， 没有意识到需要关debug, 其实tomcat一直在处理这个请求，最终导致服务器上的swap空间用尽，同事接到报警电话，所以一定要关debug,释放连接。如果真的发生了这种事情，只能重启tomcat，释放swap。用free -m 查看内存使用情况**



## 2.如何用idea导出数据库的数据为inset语句

### 其实用navicat也可以导出sql脚本，做法如下：

1.选择你要导出的表

2.右键，=》转储SQL文件=》结构和数据  或 结构

3.生成sql脚本

**但是这有个问题， 就是sql语句中的插入语句不会有字段列表**：

```java
INSERT INTO hotel_promotion_rules VALUES (155615, 20180905, 'aba_5690', 'city_url', '2018-09-04', '2018-09-10', 9999, 840, 500, '直升机', 'B类酒店');
```

### idea中的做法

1.选中你要导出的表

![选区_420.png](https://i.loli.net/2018/09/06/5b90064647e86.png)

2.右键 =》Dump Data to file =》SQL inert语句

3.保存脚本文件

-----------  **带有字段列表**

```java
INSERT INTO mobrank.hotel_promotion_rules (id, log_day, hotel_seq, city_url, start_valid_time, end_valid_time, click_num_max, exposure_num_max, expousre_num_limit, business_name, hotel_promotion_level) VALUES (155615, 20180905, 'aba_5690', 'city_url', '2018-09-04', '2018-09-10', 9999, 840, 500, '直升机', 'B类酒店');
```

4.如果想要保存一部分的数据，可以使用类似`SELECT * FROM hotel_promotion_rules LIMIT 0, 20;`查询出来后再保存成sql语句。



## 3.idea 如何使用git 操作



https://blog.csdn.net/autfish/article/details/52513465