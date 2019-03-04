# Spring 配置文件概述



[TOC]

## 1.Spring 容器启动的基本条件

- Spring的框架类包

- Bean的配置信息

- Bean的实现类

  ![image-20181117233654713](https://ws3.sinaimg.cn/large/006tNbRwly1fxbhglwicxj31g90nhgza.jpg)

- IOC(控制反转)

  那些方面的控制被反转了？

  >获得依赖对象的过程被反转了，获得依赖对象的过程由自身管理变为了由ioc容器自动注入。所以控制反转也叫做依赖注入，就是由ioc容器在运行期间，动态地将某种依赖关系注入到对象中。



- 在用 default-autowire的时候值为byname和bytype时 不能添加构造方法 在用constractor时可以用set方法 坑啊 这是为什么

> 在用 default-autowire的时候值为byname和bytype时  不是不能添加构造方法，你应该是没有添加无参的构造方法，所以才报错的。当你添加了有参的构造方法的时候，系统就不会为你添加默认的无参的构造方法了。而byName和byType的时候是需要无参构造方法的。