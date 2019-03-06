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