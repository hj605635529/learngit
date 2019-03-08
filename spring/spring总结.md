# spring总结

[TOC]



##  1. [web.xml中load-on-startup的作用](https://www.cnblogs.com/lemon-now/p/5542301.html)



我们在web.xml中配置servlet的时候会有个属性<load-on-startup></load-on-startup>，这里主要记一下它的作用，源码在后续记得好好看一下。

1. load-on-startup 元素标记容器是否应该在web应用程序启动的时候就加载这个servlet，(实例化并调用其init()方法)。
2. 它的值必须是一个整数，表示servlet被加载的先后顺序。
3. 如果该元素的值为负数或者没有设置，则容器会当Servlet被请求时再加载。
4. 如果值为正整数或者0时，表示容器在应用启动时就加载并初始化这个servlet，值越小，servlet的优先级越高，就越先被加载。值相同时，容器就会自己选择顺序来加载。

## 2. 浅谈init-param与context-param区别

https://blog.csdn.net/fengshoudong/article/details/78884349



## 3.springmvc流程总结

1.handlermapping

- 初始化阶段，扫描ioc容器中所有的bean,  看那个bean上面有controller注解， 然后通过反射拿到这个对象所有的方法，遍历这些方法， 如果方法上面有requestMapping, 则用url, 对象， 方法初始化Handler.最终保存到List<Handler>中。

2.adapther

适配器也是一个List<HandlerAdapther>, 每一个HandlerAdapther保存一个Handler, 一个map, map中保存参数类型，以及对应的索引。  初始化阶段就是保存每个handler对应的adapther。通过class对象拿到参数列表， 保存到map中

启动阶段： 通过用户的请求在handlermapping中拿到对应的handler, 拿到这个handler中保存的方法，， 解析用户的请求， 对map中保存的参数列表自动注入值， 然后通过反射去调用controller层的方法了。

3.viewResolvers

视图解析器也是一个List<ViewResolver> , 每个ViewResolver中保存视图文件名， 和视图文件。

初始化阶段就是保存某一个目录下所有的视图文件

启动阶段： 通过modleAndView中的viewName在视图解析器List中找到对应的ViewResolver.用正则表达式去处理字符串。

****

## 4. bean的初始化和销毁

关于在spring  容器初始化 bean 和销毁前所做的操作定义方式有三种：

[第一种：通过@PostConstruct 和 @PreDestroy 方法 实现初始化和销毁bean之前进行的操作](http://write.blog.csdn.net/postedit/8681497)

第二种是：[通过 在xml中定义init-method 和  destory-method方法](http://blog.csdn.net/topwqp/article/details/8681467)

第三种是：[通过bean实现InitializingBean和 DisposableBean接口](http://blog.csdn.net/topwqp/article/details/8681573)



## 5.spring父子容器

通常的使用过程中，Spring父容器对SpringMVC子容器中的bean是不可见的，而子容器对父容器的中bean却是可见的。

web.xml中的contextLoaderListener在web容器启动时，会监听到web容器的初始化事件，其contextInitialized方法会被调用，在这个方法中，spring会初始化一个启动上下文作为根上下文，即WebApplicationContext。WebApplicationContext只是接口类，其实际的实现类是XmlWebApplicationContext。
此上下文即作为Spring根容器，其对应的bean定义的配置由web.xml中的context-param中的contextConfigLocation指定。根容器初始化完毕后，Spring以
WebApplicationContext.ROOTWEBAPPLICATIONCONTEXTATTRIBUTE
为属性Key，将其存储到ServletContext中，便于获取。

再次，contextLoaderListener监听器初始化完毕后，开始初始化web.xml中配置的Servlet，这个servlet可以配置多个，以最常见的[DispatcherServlet](https://www.baidu.com/s?wd=DispatcherServlet&tn=24004469_oem_dg&rsv_dl=gh_pl_sl_csd)为例，这个servlet实际上是一个标准的前端控制器，用以转发、匹配、处理每个servlet请求。DispatcherServlet上下文在初始化的时候会建立自己的IoC上下文，用以持有spring mvc相关的bean。在建立DispatcherServlet自己的IoC上下文时，会利用WebApplicationContext.ROOTWEBAPPLICATIONCONTEXTATTRIBUTE先从ServletContext中获取之前的根上下文(即WebApplicationContext)作为自己上下文的parent上下文。有了这个parent上下文之后，再初始化自己持有的上下文。这个DispatcherServlet初始化自己上下文的工作在其initStrategies方法中可以看到，大概的工作就是初始化处理器映射、视图解析等。这个servlet自己持有的上下文默认实现类也是mlWebApplicationContext。初始化完毕后，spring以与servlet的名字相关(此处不是简单的以servlet名为Key，而是通过一些转换，具体可自行查看源码)的属性为属性Key，也将其存到ServletContext中，以便后续使用。这样每个servlet就持有自己的上下文，即拥有自己独立的bean空间，同时各个servlet共享相同的bean，即根上下文(第2步中初始化的上下文)定义的那些bean。

@Vaule注解只能在当前所在容器中获取值，所以在Controller中使用@Value注解只能在springMVC容器中取值，没法取到spring容器中加载的配置文件的值      但是这里有一个例外property-placeholder，容器中读取的配置文件就是私有的，互相不能访问。

## 6. spring如何保证创建的单例不会被jvm回收

底层实现中将所有的bean保存到一个map中， 并且这个map 被final修饰， 可以作为GCRoot, 所以可以保证创建的单例不会被jvm回收。





## 7. final关键字

https://www.cnblogs.com/lixiaolun/p/4317004.html



## 8.stringBuffer和stringBuilder





## 9 CAS

CAS存在问题：

- ABA问题：CAS要判断操作值是否发生了变化，如果操作值由A变为B，又由B变为A则CAS无法判断正确。解决ABA问题的方法是加版本号，即使值由A到B再到A，两个A的版本号不同，CAS可以判断出值是否变化。

- 循环时间长开销大：自旋CAS如果长时间不成功，会给CPU带来非常大的执行开销。

- 只能保证一个共享变量的原子操作：当对一个共享变量（volatile修饰）执行操作时，我们可以使用循环CAS的方式来保证原子操作，但是对多个共享变量操作时，循环CAS就无法保证操作的原子性，例如共享变量a和b，CAS变量a为true，CAS变量b为false但是此时a已经被替换，所以循环CAS无法保证多个共享变量的原子性，这个时候就可以用锁，或者把多个共享变量合并成一个共享变量来操作。将多个变量放到一个对象里来操作。