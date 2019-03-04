# Spring获取Bean的两种方法

[TOC]

## 问题

我们能通过ApplicationContext.getBean()和直接用注解@Resource来获取Bean,但是这两者有什么区别呢？

## 总结

ApplicationContext.getBean比较适合没有单例注解的类中使用，比如某个工具类要使用spring托管的某个类的方法，就可以通过这个来获取

