# 项目中mybatis问题排查总结

[TOC]



## 1.前提

接手一个新需求，写几个http服务接口， 原来的项目中dao层几乎都是用jdbc写的，这次想用mybatis写。

## 2.准备工作

![选区_670.png](https://i.loli.net/2018/10/19/5bc9a6c2a3d43.png)

![选区_671.png](https://i.loli.net/2018/10/19/5bc9a716bb179.png)

![选区_662.png](https://i.loli.net/2018/10/19/5bc9a75b47175.png)

原来spring-dao.xml

![选区_664.png](https://i.loli.net/2018/10/19/5bc9a7a0f1251.png)

改成：

![选区_665.png](https://i.loli.net/2018/10/19/5bc9a7d024764.png)

## 3.出现问题

``` java
org.springframework.context.annotation.ConflictingBeanDefinitionException: Annotation-specified bean name 'IHotelStatusDao' for bean class [com.qunar.mobile.web.hotel.dao.IHotelStatusDao] conflicts with existing, non-compatible bean definition of same name and class [com.qunar.mobile.web.hotel.dao_mybatis_majorHotel.IHotelStatusDao]
```

这日志表示IHotelStatusDao接口出现了两次， 查看代码， 确实出现两次 如下：

![选区_666.png](https://i.loli.net/2018/10/19/5bc9a80ee1332.png)

**为什么会出现这个问题：**

```JAVA
<property name="basePackage" value="com.qunar.mobile.web">
```

这个表示mybatis扫描web下所有的接口， 把接口对应的bean交由spring管理，它们对应的bean都是类名小写，这两个相同的接口都在web目录下，会生成两个相同的bean,因此会报出这个错误。

还会引出一个新的问题，原来web下的dao接口都有实现，此时扫描接口， 又会注册bean给spring管理。service层会出现多个bean选择问题。

![选区_672.png](https://i.loli.net/2018/10/19/5bc9a854660ca.png)

![选区_667.png](https://i.loli.net/2018/10/19/5bc9a8b50b802.png)

## 4.解决方法

- 暂时解决方法引入一个MapperFactoryBean， 但是这不是最优解法， 一引入新的mapper, 就需要写一个MapperFactoryBean

  ![选区_669.png](https://i.loli.net/2018/10/19/5bc9a8ea370a6.png)



## 5.补充

使用MyBatis时为什么Dao层不需要@Repository

http://heeexy.com/2018/04/03/MyBatis-ClasspathMapperScanner/   

MapperFactoryBean和MapperScannerConfiger 区别与联系

https://blog.csdn.net/liuxiao723846/article/details/43447441





