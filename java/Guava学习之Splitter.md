# Guava学习之Splitter

## 使用示例

以下参考：[官方文档](https://link.jianshu.com?t=https%3A%2F%2Fgithub.com%2Fgoogle%2Fguava%2Fwiki%2FStringsExplained%23splitter)。

## 1.Splitter

### 1.1概述

Java 中关于分词的工具类会有一些古怪的行为。例如：`String.split` 函数会悄悄地丢弃尾部分割符，而 `StringTokenizer` 处理5个空格字符串，结果将会什么都没有。

问题：`",a,,b,".split(",")` 的结果是什么？

1. "", "a", "", "b", ""
2.  `null`, "a", `null`, "b", `null` 
3. "a", `null`, "b"
4. "a", "b"
5. 以上都不是

正确答案是：5 以上都不是，应该是 `"", "a", "", "b"`。只有尾随的空字符串被跳过。这样的结果很令人费解。

Splitter 可以让你使用一种非常简单流畅的模式来控制这些令人困惑的行为。

```java
Splitter.on(',')
    .trimResults()
    .omitEmptyStrings()
    .split("foo,bar,,   qux");
```

以上代码将会返回 `Iterable<String>` ，包含 "foo"、 "bar"、 "qux"。一个 `Splitter`  可以通过这些来进行划分：`Pattern`、`char`、 `String`、`CharMatcher`。

如果你希望返回的是 `List` 的话，可以使用这样的代码 `Lists.newArrayList(splitter.split(string))`。

### 1.2工厂函数

| 方法                                                         | 描述                                               | 例子                                                         |
| ------------------------------------------------------------ | -------------------------------------------------- | ------------------------------------------------------------ |
| [`Splitter.on(char)`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23on-char-) | 基于特定字符划分                                   | `Splitter.on(';')`                                           |
| [`Splitter.on(CharMatcher)`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23on-com.google.common.base.CharMatcher-) | 基于某些类别划分                                   | `Splitter.on(';')`                                           |
| [`Splitter.on(String)`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23on-java.lang.String-) | 基于字符串划分                                     | `Splitter.on(CharMatcher.BREAKING_WHITESPACE)` `Splitter.on(CharMatcher.anyOf(";,."))` |
| [`Splitter.on(Pattern)`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23on-java.util.regex.Pattern-) [`Splitter.onPattern(String)`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23onPattern-java.lang.String-) | 基于正则表达式划分                                 | `Splitter.on(", ")`                                          |
| [`Splitter.fixedLength(int)`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23fixedLength-int-) | 按指定长度划分，最后部分可以小于指定长度但不能为空 | `Splitter.fixedLength(3)`                                    |

### 1.3修改器

| 方法                                                         | 描述                                                         | 例子                                                         |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ |
| [`omitEmptyStrings()`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23omitEmptyStrings--) | 移去结果中的空字符串                                         | `Splitter.on(',').omitEmptyStrings().split("a,,c,d")` 返回 `"a", "c", "d"` |
| [`trimResults()`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23trimResults--) | 将结果中的空格删除，等价于`trimResults(CharMatcher.WHITESPACE)` | `Splitter.on(',').trimResults().split("a, b, c, d")` 返回 `"a", "b", "c", "d"` |
| [`trimResults(CharMatcher)`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23trimResults-com.google.common.base.CharMatcher-) | 移除匹配字符                                                 | `Splitter.on(',').trimResults(CharMatcher.is('_')).split("_a ,_b_ ,c__")` 返回 `"a ", "b_ ", "c"` |
| [`limit(int)`](https://link.jianshu.com?t=http%3A%2F%2Fgoogle.github.io%2Fguava%2Freleases%2Fsnapshot%2Fapi%2Fdocs%2Fcom%2Fgoogle%2Fcommon%2Fbase%2FSplitter.html%23limit-int-) | 达到指定数目后停止字符串的划分                               | `Splitter.on(',').limit(3).split("a,b,c,d")` 返回 `"a", "b", "c,d"` |

## 2.Splitter.MapSplitter

以下参考：[Guava 是个风火轮之基础工具(2)](https://link.jianshu.com?t=http%3A%2F%2Fwww.importnew.com%2F15227.html)。

通过 `Splitter` 的 `withKeyValueSeparator` 方法可以获得 `MapSpliter` 对象。

`MapSpliter` 只有一个公共方法，如下所示。可以看到返回的对象是 `Map<String, String>`。

```java
public Map<String, String> split(CharSequence sequence)
```

以下代码将返回这样的 `Map`: `{"1":"2", "3":"4"}`。

```java
Splitter.on("#").withKeyValueSeparator(":").split("1:2#3:4");
```

需要注意的是，`MapSplitter` 对键值对格式有着严格的校验，下例会抛出 `java.lang.IllegalArgumentException` 异常。

```java
Splitter.on("#").withKeyValueSeparator(":").split("1:2#3:4:5"); 
```

因此，如果希望使用 `MapSplitter` 来拆分 KV 结构的字符串，需要保证键-值分隔符和键值对之间的分隔符不会称为键或值的一部分。也许是出于类似方面的考虑，MapSplitter 被加上了 `@Beta` 注解（未来不保证兼容，甚至可能会移除）。所以一般推荐使用 `JSON` 而不是 `MapJoiner` + `MapSplitter`。

##3. 源码分析

以下参考：[Guava 是个风火轮之基础工具(2)](https://link.jianshu.com?t=http%3A%2F%2Fwww.importnew.com%2F15227.html)。

`Splitter` 的实现中有十分明显的策略模式和模板模式，有各种神乎其技的方法覆盖，还有 Guava 久负盛名的迭代技巧和惰性计算。

###3.1 成员变量

`Splitter` 类有 4 个成员变量:

-  `CharMatcher trimmer`：用于描述删除 拆分结果的前后指定字符的 策略。
-  `boolean omitEmptyStrings`：用于控制是否删除 拆分结果中的 空字符串。
-  `Strategy strategy`：用于帮助实现策略模式。
-  `int limit`：用于控制拆分的结果个数。

### 3.2 策略模式

`Splitter` 可以根据字符、字符串、正则、长度还有 Guava 自己的字符匹配器 `CharMatcher` 来拆分字符串，基本上每种匹配模式的查找方法都不太一样，但是字符拆分的基本框架又是不变的，所以策略模式正好合用。

策略接口的定义很简单，就是传入一个 `Splitter` 和一个待拆分的字符串，返回一个迭代器。

```java
  private interface Strategy {
    Iterator<String> iterator(Splitter splitter, CharSequence toSplit);
  }
```

每个工厂函数创建最后都需要去调用基本的私有构造函数。这个创建过程中，主要是提供一个可以创建 `Iterator<String>` 的 `Strategy`。

```java
  private Splitter(Strategy strategy, boolean omitEmptyStrings, CharMatcher trimmer, int limit)；
```

以 `Splitter on(final CharMatcher separatorMatcher)` 创建函数为例，这里返回的是 `SplittingIterator` (它是个抽象类，继承了 `AbstractIterator`，而 `AbstractIterator` 继承了 `Iterator`）。

```java
  public static Splitter on(final CharMatcher separatorMatcher) {
    checkNotNull(separatorMatcher);
    return new Splitter(
        new Strategy() {
          @Override
          public SplittingIterator iterator(Splitter splitter, final CharSequence toSplit) {
            return new SplittingIterator(splitter, toSplit) {
              @Override
              int separatorStart(int start) {
                return separatorMatcher.indexIn(toSplit, start);
              }

              @Override
              int separatorEnd(int separatorPosition) {
                return separatorPosition + 1;
              }
            };
          }
        });
  }
```

`SplittingIterator` 需要覆盖实现 `separatorStart` 和 `separatorEnd` 两个方法才能实例化。这两个方法也是 `SplittingIterator` 用到的模板模式的重要组成。

### 3.3 惰性迭代器与模板模式

[惰性计算](https://link.jianshu.com?t=https%3A%2F%2Fbaike.baidu.com%2Fitem%2F%25E6%2583%25B0%25E6%2580%25A7%25E8%25AE%25A1%25E7%25AE%2597%2F3081216)目的是要最小化计算机要做的工作，即把计算推迟到不得不算的时候进行。Java中的惰性计算可以参考[《你应该更新的 Java 知识之惰性求值：Supplier 和 Guava》](https://link.jianshu.com?t=https%3A%2F%2Fmy.oschina.net%2Fbairrfhoinn%2Fblog%2F142985)。

Guava 中的迭代器使用了惰性计算的技巧，它不是一开始就算好结果放在列表或集合中，而是在调用 `hasNext` 方法判断迭代是否结束时才去计算下一个元素。

#### AbstractIterator

为了看懂 Guava 的惰性迭代器实现，我们要从 `AbstractIterator` 开始。

`AbstractIterator` 使用私有的枚举变量 `state` 来记录当前的迭代进度，比如是否找到了下一个元素，迭代是否结束等。`AbstractIterator` 有一个抽象方法 `computeNext`，负责计算下一个元素。由于 `state` 是私有变量，而迭代是否结束只有在调用 `computeNext` 的过程中才知道，于是提供了一个保护的 `endOfData` 方法，允许子类将 `state` 设置为 `State.DONE`。

```java
  private enum State {
    READY,
    NOT_READY,
    DONE,
    FAILED,
  }
```

`AbstractIterator` 实现了迭代器最重要的两个方法，`hasNext` 和 `next`。

`hasNext` 很容易理解，一上来先判断迭代器当前状态，如果已经结束，就返回 `false`；如果已经找到下一个元素，就返回 `true`，不然就试着找找下一个元素。

```java
  @Override
  public final boolean hasNext() {
    checkState(state != State.FAILED);
    switch (state) {
      case READY:
        return true;
      case DONE:
        return false;
      default:
    }
    return tryToComputeNext();
  }

  private boolean tryToComputeNext() {
    state = State.FAILED; // temporary pessimism
    next = computeNext();
    if (state != State.DONE) {
      state = State.READY;
      return true;
    }
    return false;
  }
```

`next` 则是先判断是否还有下一个元素，属于防御式编程，先对自己做保护；然后把状态复原到还没找到下一个元素，然后返回结果。至于为什么要把 `next` 置为 `null`，可能是帮助 JVM 回收对象。

```java
   @Override
    public final T next() {
      if (!hasNext()) {
        throw new NoSuchElementException();
      }
      state = State.NOT_READY;
      T result = next;
      next = null;
      return result;
    }
```

`tryToComputeNext` 可以认为是对模板方法 `computeNext` 的包装调用，首先把状态置为失败，然后才调用 computeNext。这样一来，如果计算下一个元素的过程中发生 `RuntimeException`，整个迭代器的状态就是 `State.FAILED`，一旦收到任何调用都会抛出异常。

```java
private boolean tryToComputeNext() {
    state = State.FAILED; // 暂时悲观
    next = computeNext();
    if (state != State.DONE) {
      state = State.READY;
      return true;
    }
    return false;
  }
```

`AbstractIterator` 的代码就这些，我们现在知道了它的子类需要覆盖实现 `computeNext` 方法，然后在迭代结束时调用 `endOfData`。接下来看看 `SplittingIterator` 的实现。

#### SplittingIterator

`SplittingIterator` 还是一个抽象类，虽然实现了 `computeNext` 方法，但是它又定义了两个抽象方法:

-  `separatorStart`： 返回分隔符在指定下标之后第一次出现的下标
-  `separatorEnd`： 返回分隔符在指定下标后面第一个不包含分隔符的下标。

之前的策略模式中我们可以看到，这两个函数在不同的策略中有各自不同的覆盖实现，在 `SplittingIterator` 中，这两个函数就是模板函数。

接下来看看 `SplittingIterator` 的核心函数 `computeNext`，这个函数一直在维护的两个内部全局变量: `offset` 和 `limit`。

```java
  @Override
    protected String computeNext() {
      // 返回的字符串介于上一个分隔符和下一个分隔符之间。
      // nextStart 是返回子串的起始位置，offset 是下次开启寻找分隔符的地方。 
      int nextStart = offset;
      while (offset != -1) {
        int start = nextStart;
        int end;

        // 找 offset 之后第一个分隔符出现的位置
        int separatorPosition = separatorStart(offset);
        if (separatorPosition == -1) {
          // 处理没找到的情况
          end = toSplit.length();
          offset = -1;
        } else {
          // 处理找到的情况
          end = separatorPosition;
          offset = separatorEnd(separatorPosition);
        }
        
        // 处理的是第一个字符就是分隔符的特殊情况
        if (offset == nextStart) {
          // 发生情况：空字符串 或者 整个字符串都没有匹配。
          // offset 需要增加来寻找这个位置之后的分隔符，
          // 但是没有改变接下来返回字符串的 start 的位置，
          // 所以此时它们二者相同。
          offset++;
          if (offset > toSplit.length()) {
            offset = -1;
          }
          continue;
        }

        // 根据 trimmer 来对找到的元素做前处理，比如去除空白符之类的。
        while (start < end && trimmer.matches(toSplit.charAt(start))) {
          start++;
        }
        // 根据 trimmer 来对找到的元素做后处理，比如去除空白符之类的。
        while (end > start && trimmer.matches(toSplit.charAt(end - 1))) {
          end--;
        }
        // 根据需要去除那些是空字符串的元素，trim完之后变成空字符串的也会被去除。
        if (omitEmptyStrings && start == end) {
          // Don't include the (unused) separator in next split string.
          nextStart = offset;
          continue;
        }

        // 判断 limit，
        if (limit == 1) {
          // The limit has been reached, return the rest of the string as the
          // final item. This is tested after empty string removal so that
          // empty Strings do not count towards the limit.
          end = toSplit.length();
          // 调整 end 指针的位置标记 offset 为 -1，下一次再调用 computeNext 
          // 的时候就发现 offset 已经是 -1 了，然后就返回 endOfData 表示迭代结束。
          offset = -1;
          // Since we may have changed the end, we need to trim it again.
          while (end > start && trimmer.matches(toSplit.charAt(end - 1))) {
            end--;
          }
        } else {
          // 还没到 limit 的极限，就让 limit 自减
          limit--;
        }

        return toSplit.subSequence(start, end).toString();
      }
      return endOfData();
    }
  }
```

## 4.对比Apache的StringUtils

首先看基本的使用方法:

```java
// Apache StringUtils...  
String[] tokens1= StringUtils.split("one,two,three",',');  
   
// Google Guava splitter...  
Iteratable<String> tokens2 = Splitter.on(','),split("one,two,three");  
```

很明显，google提供的方法更加的面向对象一点，因为它要先创建一个Splitter对象，然后使用它来分割字符串，而apache的方法则有点函数式编程的味道，它的方法都是静态的。 

这里我更加倾向于采用google的splitter，因为这个对象是可以重用的，且可以在其上附加更多的功能，比如trim，去掉空的元素等，一切都很简单。 

```java
Splitter niceCommaSplitter = Splitter.on(',') .omitEmptyString().trimResults();  
niceCommaSplitter.split("one,, two,  three"); //"one","two","three"  
niceCommaSplitter.split("  four  ,  five  "); //"four","five"
```

看起来有点用，还有其他区别么？ 
另外一个需要注意的地方就是Splitter返回的是Iteratable<String>，而StringUtils.split返回的是一个String数组。 

大部分使用分隔符的情况是我们需要对字符串按照分隔符进行遍历处理，仅此而已。 
下面就是常用的代码性能对比的例子： 

```java
@Test
public  void test2(){
    final String numberList = "One,Two,Three,Four,Five,Six,Seven,Eight,Nine,Ten";


    long start = System.currentTimeMillis();
    for(int i=0; i<1000000; i++) {
        String[] split = StringUtils.split(numberList, ',');
    }
    System.out.println(System.currentTimeMillis() - start);

    start = System.currentTimeMillis();
    for(int i=0; i<1000000; i++) {
        Iterable<String> split = Splitter.on(',').split(numberList);
    }
    System.out.println(System.currentTimeMillis() - start);


    start = System.currentTimeMillis();
    Splitter s = Splitter.on(',');
    for (int i = 0; i < 1000000; i++) {
        s.split(numberList);
    }
    System.out.println(System.currentTimeMillis() - start);
}
```
代码很简单，就是都对同一个字符串进行100万次的分隔操作，看看时间上的区别，结果如下： 

![image-20190118105919386](https://ws2.sinaimg.cn/large/006tNc79ly1fzajzg98taj30l401zq2t.jpg)

guava的速度快很多，这个程序如果运行在每天处理大量字符串的服务中，那么性能差异更加明显。我想其中的原因是Splitter返回的是Iterable<String>，而StringUtils.split返回的是一个String[]，需要创建新的String对象，导致耗时增加。 如果我们对Splitter对象缓存，那么速度提高更多

**注意：然后是网上另外一个例子：**

```java
@Test
public void tese3(){
    final String numberList = "One,Two,Three,Four,Five,Six,Seven,Eight,Nine,Ten";
    long start = System.currentTimeMillis();
    for (int i = 0; i < 10000000; i++) {
        final String[] numbers = StringUtils.split(numberList, ',');
        for (String number : numbers) {
            number.length();
        }
    }
    System.out.println(System.currentTimeMillis() - start);
    start = System.currentTimeMillis();
    for (int i = 0; i < 10000000; i++) {
        Iterable<String> is = Splitter.on(',').split(numberList);
        for(String s:is) {
            s.length();
        }
    }
    System.out.println(System.currentTimeMillis() - start);
    start = System.currentTimeMillis();
    Splitter sp = Splitter.on(',');
    for (int i = 0; i < 10000000; i++) {
        Iterable<String> is =  sp.split(numberList);
        for(String s:is) {
            s.length();
        }
    }
    System.out.println(System.currentTimeMillis() - start);
}
```

这是我测试的结果：发现还是Splitter更快， 将Splitter对象缓存， 居然都不是最快。。。。

![image-20190118110457499](https://ws3.sinaimg.cn/large/006tNc79ly1fzak5a244hj30gp01xaa0.jpg)

这是网上的实验截图：（网上用的是百万级的数据， 我用的是千万级别）

![image-20190118110628291](https://ws1.sinaimg.cn/large/006tNc79ly1fzak6upl76j30k909475e.jpg)



作者：草莓小王子

链接：https://www.jianshu.com/p/fdd4204d580a

來源：简书

简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。



https://vipcowrie.iteye.com/blog/1513693