# 深入分析Java中的length和length()

[TOC]

## 1.引子

>在不使用任何带有自动补全功能IDE的情况下，如何获取一个数组的长度？以及，如何获取一个字符串的长度？

答案如下：

```java
	@Test
	public void test(){
		int[] arr = new int[3];
		System.out.println(arr.length);  //使用length获取数组的长度

		String str = "abc";
		System.out.println(str.length()); //使用length()获取字符串的长度
	}
```

那么问题来了，为什么数组有`length`属性，而字符串没有？或者，为什么字符串有`length()`方法，而数组没有？

## 2.为什么数组有length属性

首先，数组是一个容器对象，其中包含固定数量的同一类型的值。一旦数组被创建，他的长度就是固定的了。数组的长度可以作为`final`实例变量的长度。因此，长度可以被视为一个数组的属性。

有两种创建数组的方法：

1、通过数组表达式创建数组。

2、通过初始化值创建数组。

无论使用哪种方式，一旦数组被创建，其大小就固定了。

使用表达式创建数组方式如下，该方式指明了元素类型、数组的维度、以及至少一个维度的数组的长度。

```
int[][] arr = new int[3][];
```

该声明方式是符合要求的，因为他指定了一个维度的长度（该数组的类型为int，维度为2，第一维度的长度为3）

使用数组初始化的方式创建数组时需要提供所有的初始值。形式是使用`{`}`将所有初始值括在一起并用`,隔开。

```
int[] arr = {1,2,3};
```

注：

这里可能会有一个疑问，既然数组大小是初始化时就规定好的，那么`int[][] arr = new int[3][];`定义的数组并没有给出数组的第二维的大小，那么这个`arr`的长度到底是如何“规定好”的呢？

其实，`arr`的长度就是3。其实Java中所有的数组，无论几维，其实都是一维数组。例如arr，分配了3个空间，每个空间存放一个一维数组的地址，这样就成了“二维”数组。但是对于arr来说，他的长度就是3。

```java
@Test
	public void test2(){
		int[][] a = new int[3][];;
		System.out.println(a.length);    //3

		int[][] b = new int[3][5];
		System.out.println(b.length);  //3
	}
```

## 3.为什么String有length()方法

String背后的数据结构是一个char数组,所以没有必要来定义一个不必要的属性（因为该属性在char数值中已经提供了）

注：要想把char[]转成字符串有以下方式：

```java
	char[] s = {'a','b','c'};
		String s1 = s.toString();
		String s2 = String.valueOf(s);
		String s3 = new String(s);
```

**为啥没有必要给String类添加length属性？**

这里就为什么没有必要说说自己的一点看法。 在通常的java应用中都存在大量的String对象，如果在String类中添加length属性，那么每个String对象都必须多出一部分空间去存储length，会有大量的空间被占用。而如果不提供的话，当要获得String长度的时候必须先在String对象中找到char数组的引用地址，然后访问该地址上的数组对象拿到数组长度，比直接访问String对象多了一次调用。但是综合考虑现实情况:应用程序中有很多String对象，但是需要得到length的String对象占少数，综合空间占用和性能考虑，String设计者提供了length()方法。


