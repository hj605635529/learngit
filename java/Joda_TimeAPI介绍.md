# Joda Time API 介绍

[TOC]

## 1 简介

在Java中处理日期和时间是很常见的需求，基础的工具类就是我们熟悉的Date和Calendar，然而这些工具类的api使用并不是很方便和强大，于是就诞生了Joda-Time这个专门处理日期时间的库。

## 2 引入maven依赖

```java
<dependency>
    <groupId>joda-time</groupId>
    <artifactId>joda-time</artifactId>
    <version>2.9.2</version>
</dependency>
```

## 3 核心类介绍

下面介绍5个最常用的date-time类：

- Instant - 不可变的类，用来表示时间轴上一个瞬时的点
- DateTime - 不可变的类，用来替换JDK的Calendar类
- LocalDate - 不可变的类，表示一个本地的日期，而不包含时间部分（没有时区信息）
- LocalTime - 不可变的类，表示一个本地的时间，而不包含日期部分（没有时区信息）
- LocalDateTime - 不可变的类，表示一个本地的日期－时间（没有时区信息）

>注意：不可变的类，表明了正如Java的String类型一样，其对象是不可变的。即，不论对它进行怎样的改变操作，返回的对象都是新对象。

 **DateTime的主要目的是替换JDK中的Calendar类，用来处理那些时区信息比较重要的场景**

 **LocalDate比较适合表示出生日期这样的类型，因为不关心这一天中的时间部分**

 **LocalTime适合表示一个商店的每天开门/关门时间，因为不用关心日期部分**

```java
//获取一个本地的日期，没有时间部分
LocalDate localDate = new LocalDate();
System.out.println(localDate);

//获取一个本地的时间，不包含日期部分
LocalTime localTime = new LocalTime();
System.out.println(localTime);
```

结果：

```java
2018-08-26
21:24:43.133
```



## 4 DateTime类介绍

### 4.1 与java.util.Date相互转换

```java
java.util.Date date = new Date();

//java.util.Date -> Joda DateTime
DateTime dateTime = new DateTime(date);

//Joda DateTime -> java.util.Date
Date date2 = dateTime.toDate();

System.out.println(date2);
```

 ### 4.2  将字符串格式转换成DateTime格式

```java
//1.将字符串格式转换成DateTime格式， 注意这里是DateTimeFormat不是DateTimeFormatter
DateTime parse = DateTime.parse("2018-08-26", DateTimeFormat.forPattern("yyyy-MM-dd"));
//2.这个也能转换
DateTime fromDate = DateTimeFormat.forPattern("yyyy-MM-dd").parseDateTime(fromDateStr);
System.out.println(parse);
```

### 4.3 将DateTime按照某种格式输出

```java
//5.将DateTime按照某种格式输出
DateTime now1 = DateTime.now();
System.out.println(now1.toString("yyyy-MM-dd HH:mm:ss"));
```

 ### 4.4 获取年月日时分秒毫秒

```java
DateTime dt = new DateTime();
//获取当前时间的年
int year = dt.getYear();
//获取当前时间的月
int month = dt.getMonthOfYear();
//获取当前时间是一年中的第几天
nt dayOfYear = dt.getDayOfYear();
//获取一个月中的天
int day =  dt.getDayOfMonth();
//获取一周中的周几
int week = dt.getDayOfWeek();
//一天中的第几小时(取整)
int hour = dt.getHourOfDay();
//当前时间的秒中的毫秒 
int ms = dt.getMillisOfSecond();
//获取当前时间的秒
int second = dt.getSecondOfDay();  
 //获取当前时间的毫秒
 long millis = dt.getMillis();
```

### 4.4 计算两个日期相差几天

```java
DateTime date1 = DateTime.parse("2018-08-26", DateTimeFormat.forPattern("yyyy-MM-dd"));
DateTime date2 = DateTime.parse("2017-09-23", DateTimeFormat.forPattern("yyyy-MM-dd"));
//计算date2相差date1多少天
System.out.println(Days.daysBetween(date2, date1).getDays());
```

 ### 4.5 判断一个日期是否在日期之前或之后

```java
DateTime date1 = DateTime.parse("2018-08-26", DateTimeFormat.forPattern("yyyy-MM-dd"));
DateTime date2 = DateTime.parse("2017-09-23", DateTimeFormat.forPattern("yyyy-MM-dd"));

System.out.println(date1.isAfter(date2));
System.out.println(date1.isBefore(date2));
```

### 4.6 获取当天的日期 

```java
//获取当天的日期， 时间部分都是0
DateTime dateTime = DateTime.now().withTime(0, 0, 0, 0);
System.out.println(dateTime);
```

 ### 4.7 当前日期加上天数和月数

```java
DateTime date1 = DateTime.parse("2018-08-26", DateTimeFormat.forPattern("yyyy-MM-dd"));
//当前时间加上10天
DateTime dateTime1 = date1.plusDays(10);
System.out.println(dateTime1);
//当前时间加上1个月
DateTime dateTime2 = date1.plusMonths(1);
System.out.println(dateTime2);
```

### 4.8 是否润年

```java
DateTime date1 = DateTime.parse("2018-08-26", DateTimeFormat.forPattern("yyyy-MM-dd"));
boolean leap = date1.year().isLeap();
System.out.println("是否闰年："+leap);
```

### 4.9 时间判断是否在某个范围

```java
DateTime date1 = DateTime.parse("2018-08-26", DateTimeFormat.forPattern("yyyy-MM-dd"));
DateTime date2 = DateTime.parse("2017-09-23", DateTimeFormat.forPattern("yyyy-MM-dd"));
DateTime date3 = DateTime.parse("2017-09-23", DateTimeFormat.forPattern("yyyy-MM-dd"));


Interval interval = new Interval(date2, date1);
System.out.println(interval.contains(date3)); // true      左闭右开区间
```


spliterator