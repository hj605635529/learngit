

# java注解的应用

[TOC]

##java注解介绍

Java注解是附加在代码中的一些元信息，用于编译和运行时进行解析和使用，起到说明、配置的功能。注解不会影响代码的实际逻辑，仅仅起到辅助性的作用。包含在java.lang.annotation包中。注解的定义类似于接口的定义，使用@interface来定义，定义一个方法即为注解类型定义了一个元素，**方法的声明不允许有参数或throw语句，返回值类型被限定为原始数据类型、字符串String、Class、enums、注解类型，或前面这些的数组，方法可以有默认值**。注解并不直接影响代码的语义，但是他可以被看做是程序的工具或者类库。它会反过来对正在运行的程序语义有所影响。注解可以从源文件、class文件或者在运行时通过反射机制多种方式被读取。下面是一个例子：

```java
@Test("test")
public class AnnotationTest {
    public void test(){
    }
}
```

> 在我们的AnnotationTest类被编译后，在对应的AnnotationTest.class文件中会包含一个RuntimeVisibleAnnotations属性，由于这个注解是作用在类上，所以此属性被添加到类的属性集上。即Test注解的键值对value=test会被记录起来。而当JVM加载AnnotationTest.class文件字节码时，就会将RuntimeVisibleAnnotations属性值保存到AnnotationTest的Class对象中，于是就可以通过AnnotationTest.class.getAnnotation(Test.class)获取到Test注解对象，进而再通过Test注解对象获取到Test里面的属性值。



> 注解被编译后的本质就是一个继承Annotation接口的接口，所以@Test其实就是“public interface Test extends Annotation”，当我们通过AnnotationTest.class.getAnnotation(Test.class)调用时，JDK会通过动态代理生成一个实现了Test接口的对象，并把将RuntimeVisibleAnnotations属性值设置进此对象中，此对象即为Test注解对象，通过它的value()方法就可以获取到注解值。

##元注解

元注解的作用就是负责注解其他注解。Java5.0定义了4个标准的meta-annotation类型，它们被用来提供对其它 annotation类型作说明。Java5.0定义的元注解：

- @Target
- @Retention
- @Documented
- @Inherited

这些类型和它们所支持的类在java.lang.annotation包中可以找到。下面我们看一下每个元注解的作用和相应分参数的使用说明。

###@Target

@Target说明了Annotation所修饰的对象范围：Annotation可被用于 packages、types（类、接口、枚举、Annotation类型）、类型成员（方法、构造方法、成员变量、枚举值）、方法参数和本地变量（如循环变量、catch参数）。在Annotation类型的声明中使用了target可更加明晰其修饰的目标。
取值(ElementType)有：

- CONSTRUCTOR:用于描述构造器
- FIELD:用于描述域
- LOCAL_VARIABLE:用于描述局部变量
- METHOD:用于描述方法
- PACKAGE:用于描述包
- PARAMETER:用于描述参数
- TYPE:用于描述类、接口(包括注解类型) 或enum声明

使用实例：

```
@Target(ElementType.TYPE)
public @interface Table {
    public String tableName() default "className";
}

@Target(ElementType.FIELD)
public @interface NoDBColumn {
}
```

上述注解Table 可以用于注解类、接口(包括注解类型) 或enum声明,而注解NoDBColumn仅可用于注解类的成员变量。

### @Retention

@Retention 表示需要在什么级别保存该注解信息。可选的RetentionPolicy参数包括：

- SOURCE：注解将被编译器丢弃
- CLASS：注解在class文件中可用，但会被JVM丢弃
- RUNTIME：JVM将在运行期间保留注解，因此可以通过反射机制读取注解的信息。

Retention meta-annotation类型有唯一的value作为成员，它的取值来自java.lang.annotation.RetentionPolicy的枚举类型值。具体实例如下：

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
public @interface Column {
    public String name() default "fieldName";
    public String setFuncName() default "setField";
    public String getFuncName() default "getField";
    public boolean defaultDBValue() default false;
}
```

Column注解的的RetentionPolicy的属性值是RUTIME,这样注解处理器可以通过反射，获取到该注解的属性值，从而去做一些运行时的逻辑处理 。

### @Documented

@Documented 表示含有该注解类型的元素(带有注释的)会通过javadoc或类似工具进行文档化，Documented是一个标记注解（类似@Override 这种只需要一个简单的声明即可的注解即为标记注解），没有成员。该类型应用于注解那些影响客户使用带注释(comment)的元素声明的类型。如果类型声明是用@Documented 来注解的，这种类型的注解被作为被标注的程序成员的公共API。
实例如下：

```
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface Column {
    public String name() default "fieldName";
    public String setFuncName() default "setField";
    public String getFuncName() default "getField";
    public boolean defaultDBValue() default false;
}
```

### @Inherited

@Inherited 元注解是一个标记注解，@Inherited阐述了某个被标注的类型是被继承的。如果一个使用了@Inherited修饰的annotation类型被用于一个class，则这个annotation将被用于该class的子类。
**注意：@Inherited annotation类型是被标注过的class的子类所继承。类并不从它所实现的接口继承annotation，方法并不从它所重载的方法继承annotation。当@Inherited annotation类型标注的annotation的Retention是RetentionPolicy.RUNTIME，则反射API增强了这种继承性。如果我们使用java.lang.reflect去查询一个@Inherited annotation类型的annotation时，反射代码检查将展开工作：检查class和其父类，直到发现指定的annotation类型被发现，或者到达类继承结构的顶层。**
实例如下：

```
@Inherited
public @interface Greeting {
    public enum FontColor{ BULE,RED,GREEN};
    String name();
    FontColor fontColor() default FontColor.GREEN;
}
```

### 注解元素的默认值

注解元素必须有确定的值，要么在定义注解的默认值中指定，要么在使用注解时指定，非基本类型的注解元素的值不可为null。因此, 使用空字符串或0作为默认值是一种常用的做法。这个约束使得处理器很难表现一个元素的存在或缺失的状态，因为每个注解的声明中，所有元素都存在，并且都具有相应的值，为了绕开这个约束，我们只能定义一些特殊的值，例如空字符串或者负数，一次表示某个元素不存在，在定义注解时，这已经成为一个习惯用法。

### 实现自定义注解

使用@interface自定义注解时，自动继承了java.lang.annotation.Annotation接口，由编译程序自动完成其他细节。在定义注解时，不能继承其他的注解或接口。**@interface用来声明一个注解，其中的每一个方法实际上是声明了一个配置参数。方法的名称就是参数的名称，返回值类型就是参数的类型（返回值类型只能是基本类型、Class、String、enum）。**可以通过default来声明参数的默认值。
**定义注解格式：** 
　　*public @interface 注解名 {定义体}* 
注解参数的可支持数据类型：

- 所有基本数据类型（int,float,boolean,byte,double,char,long,short)
- String类型
- Class类型
- enum类型
- Annotation类型
- 以上所有类型的数组

**Annotation类型里面的参数该怎么设定：**

- 只能用public或默认(default)这两个访问权修饰.例如,String value();这里把方法设为defaul默认类型；　 　
- 参数成员只能用基本类型byte,short,char,int,long,float,double,boolean八种基本数据类型和 String,Enum,Class,annotations等数据类型,以及这一些类型的数组.例如,String value();这里的参数成员就为String;　　
- 如果**只有一个参数成员,最好把参数名称设为”value”，后加小括号。注解在只有一个元素且该元素的名称是value的情况下，在使用注解的时候可以省略“value=”，直接写需要的值即可**。

##常用的注解

- @Slf4j的作用：如果不想每次都写private  final Logger logger = LoggerFactory.getLogger(XXX.class); 可以用注解@Slf4j

  注意：如果注解@Slf4j注入后找不到变量log，那就给IDE安装lombok插件，具体见：https://www.cnblogs.com/weiapro/p/7633645.html

* @Deprecated    这个元素是用来标记过时的元素，想必大家在日常开发中经常碰到。编译器在编译阶段遇到这个注解时会发出提醒警告，告诉开发者正在调用一个过时的元素比如过时的方法、过时的类、过时的成员变量。

- @SuppressWarnings   阻止警告的意思。之前说过调用被 @Deprecated 注解的方法后，编译器会警告提醒，而有时候开发者会忽略这种警告，他们可以在调用的地方通过 @SuppressWarnings 达到目的。
- @FunctionalInterfac    函数式接口注解，接口只能有一个抽象的方法，比如Runable接口，如果有两个抽象方法，会报错。这个是 Java 1.8 版本引入的新特性。函数式编程很火，所以 Java 8 也及时添加了这个特性。





## 代码实例

一：

```java
/**
* 水果名称注解
*/
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitName {
    String value() default "";
}

/**
* 水果颜色注解
*/
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitColor {

    public enum Color{ BULE,RED,GREEN};

    Color fruitColor() default Color.GREEN;
}

/**
* 水果供应者注解
*/
@Target(ElementType.FIELD)
@Retention(RetentionPolicy.RUNTIME)
@Documented
public @interface FruitProvider {
    /**
     * 供应商编号
     */
    public int id() default -1;

    /**
     * 供应商名称
     */
    public String name() default "";

    /**
     * 供应商地址
     */
    public String address() default "";
}

/***********注解使用***************/

public class Apple {

    @FruitName("Apple")
    private String appleName;

    @FruitColor(fruitColor=Color.RED)
    private String appleColor;

    @FruitProvider(id=1,name="陕西红富士集团",address="陕西省西安市延安路89号红富士大厦")
    private String appleProvider;

    public void setAppleColor(String appleColor) {
        this.appleColor = appleColor;
    }
    public String getAppleColor() {
        return appleColor;
    }

    public void setAppleName(String appleName) {
        this.appleName = appleName;
    }
    public String getAppleName() {
        return appleName;
    }

    public void setAppleProvider(String appleProvider) {
        this.appleProvider = appleProvider;
    }
    public String getAppleProvider() {
        return appleProvider;
    }

    public void displayName(){
        System.out.println("水果的名字是：苹果");
    }
}

/***********注解处理器***************/

public class FruitInfoUtil {
    public static void getFruitInfo(Class clazz){

        String strFruitName=" 水果名称：";
        String strFruitColor=" 水果颜色：";
        String strFruitProvicer="供应商信息：";

        Field[] fields = clazz.getDeclaredFields();

        for(Field field :fields){
            //这里的isAnnotationPresent和getAnnotation方法都是AccessibleObject中的，Field和Method都继承这个类
            if(field.isAnnotationPresent(FruitName.class)){
                FruitName fruitName = (FruitName) field.getAnnotation(FruitName.class);
                strFruitName=strFruitName+fruitName.value();
                System.out.println(strFruitName);
            }
            else if(field.isAnnotationPresent(FruitColor.class)){
                FruitColor fruitColor= (FruitColor) field.getAnnotation(FruitColor.class);
                strFruitColor=strFruitColor+fruitColor.fruitColor().toString();
                System.out.println(strFruitColor);
            }
            else if(field.isAnnotationPresent(FruitProvider.class)){
                FruitProvider fruitProvider= (FruitProvider) field.getAnnotation(FruitProvider.class);
                strFruitProvicer=" 供应商编号："+fruitProvider.id()+" 供应商名称："+fruitProvider.name()+" 供应商地址："+fruitProvider.address();
                System.out.println(strFruitProvicer);
            }
        }
    }
}

/***********输出结果***************/
public class FruitRun {

    /**
     * @param args
     */
    public static void main(String[] args) {

        FruitInfoUtil.getFruitInfo(Apple.class);

    }

}
====================================
水果名称：Apple
水果颜色：RED
供应商编号：1 供应商名称：陕西红富士集团 供应商地址：陕西省西安市延安路89号红富士大厦
```
二：

注解的定义：

```java
@Documented
@Inherited
@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.METHOD})
public @interface SearchDataTask {

    String name() default "";

    //是否需要初始化时调用
    boolean isNeedInit() default true;
    
    //是否有schedule
    boolean hasSchedule() default true;

    long initialDelay() default -1L;

    long fixedDelay() default -1L;
}
```

注解处理器：

```java
@Slf4j
public class AutomaticTaskScanner implements BeanPostProcessor, DisposableBean {


    private List<AutomaticTaskInfo> initList = Lists.newArrayList();

    List<AutomaticTaskInfo> getInitList() {
        return initList;
    }

    private AtomicBoolean isInitFinish = new AtomicBoolean(false);


    void setIsInitFinish() {
        isInitFinish.set(true);
    }

    private ScheduledExecutorService scheduledExecutorService = Executors.newScheduledThreadPool(5);

    /**
     * @param o    对应的bean元素，也就是spring管理的对象
     * @param s    bean元素的名字，也就是spring管理对象的名字
     * 在bean元素初始化之前执行，记住最后一定需要返回bean
     */
    @Override
    public Object postProcessBeforeInitialization(Object o, String s) throws BeansException {
        parseMethods(o, o.getClass().getDeclaredMethods());
        return o;
    }

    /**
     * @param o  同上
     * @param s  同上
     */
    @Override
    public Object postProcessAfterInitialization(Object o, String s) throws BeansException {
        return o;
    }


    /**
     * @param o
     * @param methods
     */
    private void parseMethods(Object o, Method[] methods) {
        for (Method method : methods) {
            //找到方法上的SearchDataTask注解 这里用了spring提供的AnnotationUtils工具类
            SearchDataTask searchDataTask = AnnotationUtils.findAnnotation(method, SearchDataTask.class);
            if (searchDataTask == null) {
                continue;
            }
            if (searchDataTask.isNeedInit()) {
           	 initList.add(AutomaticTaskInfo.builder().bean(o).method(method).build());
            }
            log.info("add method:{}", method.toString());
            if (searchDataTask.hasSchedule()) {
                //用了lambda表达式
                scheduledExecutorService.scheduleAtFixedRate(() -> {
                    try {
                        if (isInitFinish.get()) {
                            log.info("schedule:{}", method.toString());
                            method.invoke(o);
                        }
                    } catch (IllegalAccessException | InvocationTargetException e) {
                        log.error("error:{}", method.toString(), e);
                    }
                }, searchDataTask.initialDelay(), searchDataTask.fixedDelay(), TimeUnit.MINUTES);
            }
        }
    }

    //定时任务的关闭
    @Override
    public void destroy() {
        if (scheduledExecutorService != null) {
            scheduledExecutorService.shutdown();
        }
    }
}
```
使用：

```java
@SearchDataTask(initialDelay = 6, fixedDelay = 60)
public void update() {
    try {
        StopWatch stopWatch = new StopWatch();
        stopWatch.start();
        Integer maxVersion = hotelOfficialScoreDao.selectMaxVersion();
        if (maxVersion != null && maxVersion > dataVersion.get()) {
            List<HotelOfficialScore> data = hotelOfficialScoreDao.selectOfficialScore(maxVersion);
            for (HotelOfficialScore item : data) {
                String hotelSeq = item.getHotelSeq();
                hotelOfficialScoreMap.put(hotelSeq,item);
            }
        }
        dataVersion.set(maxVersion.intValue());
        stopWatch.stop();
        LOGGER.info("获取官方评分和入住人数，用时{} ms.", stopWatch.getTime());
    } catch (Exception e) {
        QMonitor.recordOne("search_official_score_error");
        LOGGER.error("获取官方评分和入住人数", e);
    }
}
```