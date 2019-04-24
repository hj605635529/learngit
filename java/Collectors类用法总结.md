# Collectors用法总结

[TOC]

## 0.前提

对流调用collect方法将对流中的元素触发一个归约操作(由Collector来参数化)

```java
<R, A> R collect(Collector<? super T, A, R> collector);

 <R> R collect(Supplier<R> supplier,
                  BiConsumer<R, ? super T> accumulator,
                  BiConsumer<R, R> combiner);
```

Collector接口中方法的实现决定了如何对流执行归约操作。但Collectors实用类提供了很多静态工厂方法， 可以方便地创建常见收集器的实例，只要拿来用就可以了。最直接和最常用的收集器是toList 静态方法，它会把流中所有的元素收集到一个List中: 

```java
    List<Transaction> transactions =
        transactionStream.collect(Collectors.toList());
```

Collectors类主要提供了三大功能:

- 1.将流元素归约和汇总成一个值。
- 2.元素分组
- 3.元素分区

## 1.归约和汇总



### 1.1特殊的归约与汇总

```java
@Data
class Dish {

    private final String name;
    private final boolean vegetarian;
    private final int calories;
    private final Type type;

    public Dish(String name, boolean vegetarian, int calories, Type type) {
        this.name = name;
        this.vegetarian = vegetarian;
        this.calories = calories;
        this.type = type;
    }

    public enum Type { MEAT, FISH, OTHER }


    public static final List<Dish> menu =
            Arrays.asList( new Dish("pork", false, 800, Dish.Type.MEAT),
                    new Dish("beef", false, 700, Dish.Type.MEAT),
                    new Dish("chicken", false, 400, Dish.Type.MEAT),
                    new Dish("french fries", true, 530, Dish.Type.OTHER),
                    new Dish("rice", true, 350, Dish.Type.OTHER),
                    new Dish("season fruit", true, 120, Dish.Type.OTHER),
                    new Dish("pizza", true, 550, Dish.Type.OTHER),
                    new Dish("prawns", false, 400, Dish.Type.FISH),
                    new Dish("salmon", false, 450, Dish.Type.FISH));

    public static final Map<String, List<String>> dishTags = Maps.newHashMap();

    static {
        dishTags.put("pork", Arrays.asList("greasy", "salty"));
        dishTags.put("beef", Arrays.asList("salty", "roasted"));
        dishTags.put("chicken", Arrays.asList("fried", "crisp"));
        dishTags.put("french fries", Arrays.asList("greasy", "fried"));
        dishTags.put("rice", Arrays.asList("light", "natural"));
        dishTags.put("season fruit", Arrays.asList("fresh", "natural"));
        dishTags.put("pizza", Arrays.asList("tasty", "salty"));
        dishTags.put("prawns", Arrays.asList("tasty", "roasted"));
        dishTags.put("salmon", Arrays.asList("delicious", "fresh"));
    }


    public static void test22(){
        //统计数量
        long count = menu.stream().count();
        System.out.println(count);
        count = menu.stream().collect(Collectors.counting());
        System.out.println(count);

        //查找最大值和最小值
        Optional<Dish> max = menu.stream().max(Comparator.comparingInt(Dish::getCalories));
        System.out.println(max.get());
        max = menu.stream().collect(Collectors.maxBy(Comparator.comparingInt(Dish::getCalories)));
        System.out.println(max);

        //求和
        Integer collect = menu.stream().collect(Collectors.summingInt(Dish::getCalories));
        System.out.println(collect);
        int sum = menu.stream().mapToInt(Dish::getCalories).sum();
        System.out.println(sum);

        //求平均数
        Double collect1 = menu.stream().collect(Collectors.averagingInt(Dish::getCalories));
        System.out.println(collect1);
        OptionalDouble average = menu.stream().mapToInt(Dish::getCalories).average();
        System.out.println(average);

        //连接字符串
        String collect2 = menu.stream().map(Dish::getName).collect(Collectors.joining("---"));
        System.out.println(collect2);
    }
}
```

![image-20190425005358783](https://ws4.sinaimg.cn/large/006tNc79gy1g2e7lij4hmj30pa05bwf6.jpg)

### 1.2 广义的归约与汇总

```java
//reducing
//第一个参数为初始值，第二个参数表示对对象中的某一个属性，第三个参数表示做什么操作
Integer collect3 = menu.stream().collect(Collectors.reducing(0, Dish::getCalories, (a, b) -> a + b));
System.out.println(collect3);
Integer reduce = menu.stream().map(Dish::getCalories).reduce(0, (a, b) -> a + b);
System.out.println(reduce);
```

![image-20190425012046670](https://ws4.sinaimg.cn/large/006tNc79gy1g2e8dchq7xj30tx01gdfn.jpg)







