# spring源码解读一

  

![ClassPathXmlApplicationContext的副本](https://ws4.sinaimg.cn/large/006tNc79ly1fywxsoe9k4j32470u0dih.jpg)





![DefaultListableBeanFactory](https://ws2.sinaimg.cn/large/006tNc79ly1fywy75jejtj315a0psjsl.jpg)





```java
public interface BeanFactory {

   /**
    * 对factoryBean的转义定义， 因为如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象，
    * 如果需要得到工厂本身， 需要转义
    */
   String FACTORY_BEAN_PREFIX = "&";
  /**
   *根据bean的名字，获取在ioc容器中得到的bean实例
   *
   */
   Object getBean(String name) throws BeansException;

  /**
   * 根据bean的名字和class类型来得到bean实例，增加了类型安全验证机制
   */
   <T> T getBean(String name, Class<T> requiredType) throws BeansException;

   /**
    * Return the bean instance that uniquely matches the given object type, if any.
    * <p>This method goes into {@link ListableBeanFactory} by-type lookup territory
    * but may also be translated into a conventional by-name lookup based on the name
    * of the given type. For more extensive retrieval operations across sets of beans,
    * use {@link ListableBeanFactory} and/or {@link BeanFactoryUtils}.
    * @param requiredType type the bean must match; can be an interface or superclass.
    * {@code null} is disallowed.
    * @return an instance of the single bean matching the required type
    * @throws NoSuchBeanDefinitionException if no bean of the given type was found
    * @throws NoUniqueBeanDefinitionException if more than one bean of the given type was found
    * @throws BeansException if the bean could not be created
    * @since 3.0
    * @see ListableBeanFactory
    */
   <T> T getBean(Class<T> requiredType) throws BeansException;

   /**
    * Return an instance, which may be shared or independent, of the specified bean.
    * <p>Allows for specifying explicit constructor arguments / factory method arguments,
    * overriding the specified default arguments (if any) in the bean definition.
    * @param name the name of the bean to retrieve
    * @param args arguments to use when creating a bean instance using explicit arguments
    * (only applied when creating a new instance as opposed to retrieving an existing one)
    * @return an instance of the bean
    * @throws NoSuchBeanDefinitionException if there is no such bean definition
    * @throws BeanDefinitionStoreException if arguments have been given but
    * the affected bean isn't a prototype
    * @throws BeansException if the bean could not be created
    * @since 2.5
    */
   Object getBean(String name, Object... args) throws BeansException;

   /**
    * Return an instance, which may be shared or independent, of the specified bean.
    * <p>Allows for specifying explicit constructor arguments / factory method arguments,
    * overriding the specified default arguments (if any) in the bean definition.
    * <p>This method goes into {@link ListableBeanFactory} by-type lookup territory
    * but may also be translated into a conventional by-name lookup based on the name
    * of the given type. For more extensive retrieval operations across sets of beans,
    * use {@link ListableBeanFactory} and/or {@link BeanFactoryUtils}.
    * @param requiredType type the bean must match; can be an interface or superclass.
    * {@code null} is disallowed.
    * @param args arguments to use when creating a bean instance using explicit arguments
    * (only applied when creating a new instance as opposed to retrieving an existing one)
    * @return an instance of the bean
    * @throws NoSuchBeanDefinitionException if there is no such bean definition
    * @throws BeanDefinitionStoreException if arguments have been given but
    * the affected bean isn't a prototype
    * @throws BeansException if the bean could not be created
    * @since 4.1
    */
   <T> T getBean(Class<T> requiredType, Object... args) throws BeansException;


   /**
    * 提供对bean的检索， 看看是否ioc容器中是否有这个名字的bean
    */
   boolean containsBean(String name);

   /**
    * 根据bean名字得到bean实例， 并同时判断这个bean是否是单例
    */
   boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

   /**
    * Is this bean a prototype? That is, will {@link #getBean} always return
    * independent instances?
    * <p>Note: This method returning {@code false} does not clearly indicate
    * a singleton object. It indicates non-independent instances, which may correspond
    * to a scoped bean as well. Use the {@link #isSingleton} operation to explicitly
    * check for a shared singleton instance.
    * <p>Translates aliases back to the corresponding canonical bean name.
    * Will ask the parent factory if the bean cannot be found in this factory instance.
    * @param name the name of the bean to query
    * @return whether this bean will always deliver independent instances
    * @throws NoSuchBeanDefinitionException if there is no bean with the given name
    * @since 2.0.3
    * @see #getBean
    * @see #isSingleton
    */
   boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

   /**
    * Check whether the bean with the given name matches the specified type.
    * More specifically, check whether a {@link #getBean} call for the given name
    * would return an object that is assignable to the specified target type.
    * <p>Translates aliases back to the corresponding canonical bean name.
    * Will ask the parent factory if the bean cannot be found in this factory instance.
    * @param name the name of the bean to query
    * @param typeToMatch the type to match against (as a {@code ResolvableType})
    * @return {@code true} if the bean type matches,
    * {@code false} if it doesn't match or cannot be determined yet
    * @throws NoSuchBeanDefinitionException if there is no bean with the given name
    * @since 4.2
    * @see #getBean
    * @see #getType
    */
   boolean isTypeMatch(String name, ResolvableType typeToMatch) throws NoSuchBeanDefinitionException;

   /**
    * Check whether the bean with the given name matches the specified type.
    * More specifically, check whether a {@link #getBean} call for the given name
    * would return an object that is assignable to the specified target type.
    * <p>Translates aliases back to the corresponding canonical bean name.
    * Will ask the parent factory if the bean cannot be found in this factory instance.
    * @param name the name of the bean to query
    * @param typeToMatch the type to match against (as a {@code Class})
    * @return {@code true} if the bean type matches,
    * {@code false} if it doesn't match or cannot be determined yet
    * @throws NoSuchBeanDefinitionException if there is no bean with the given name
    * @since 2.0.1
    * @see #getBean
    * @see #getType
    */
   boolean isTypeMatch(String name, Class<?> typeToMatch) throws NoSuchBeanDefinitionException;

   /**
    * 得到bean实例的class类型
    */
   Class<?> getType(String name) throws NoSuchBeanDefinitionException;

   /**
    * 得到bean的别名，如果根据别名检索， 那么其原名也会被检索出来
    */
   String[] getAliases(String name);

}
```





Spring提供许多IOC容器的实现。比如xmlBeanFactory, ClassPathXmlApplicationContext等。其中XmlBeanFactory就是针对最基本的Ioc容器的实现， 这个ioc容器可以读取xml文件定义的BeanDefinition(xml文件中对bean的描述)， 如果说xmlBeanfactory是容器中的屌丝， applicationContext应该是容器中的高富帅。

applicationContext是spring提供的一个高级的ioc容器，它除了能够提供ioc容器的基本功能外， 还为用户提供了以下的服务。

从ApplicationContext接口的实现，我们看出其特点：

1.支持信息源， 可以实现国际化（实现MessageSource接口）

2.访问资源（实现ResourcePatternResolver接口）

3.支持应用事件。（实现ApplicationEventPublisher接口）









springioc容器管理我们定义的各种bean对象及其相互的关系，bean对象在spring实现中是以beanDefinition来描述的，其继承体系如下：

![image-20190106170501030](https://ws1.sinaimg.cn/large/006tNc79ly1fywz47tvp7j31fa0u00wf.jpg)

 





IOC容器的初始化

1.定位 ：想办法去找到我们xml配置文件 得到一个Resource

2.加载: 读取xml配置文件的内容， 解析成一个Beanfinition

3.注册: 根据Beanfinition内容通过factory去new对象

我们以ApplicationContext为例讲解，ApplicationContext系列容器是我们最熟悉的， web项目中使用的xmlwebapplicationContext就是属于这个继承体系， 还有ClasspathXmlApplicationContext等，其继承体系如下：

![image-20190106172856004](https://ws3.sinaimg.cn/large/006tNc79ly1fywzt3x2qxj31bu0u00z2.jpg)

![image-20190106174841527](https://ws3.sinaimg.cn/large/006tNc79ly1fyx0dxldz9j31fc0pg77t.jpg)





![image-20190106175506481](https://ws3.sinaimg.cn/large/006tNc79ly1fyx0kc18j4j311h0hbqgm.jpg)

 ![image-20190106180011166](https://ws3.sinaimg.cn/large/006tNc79ly1fyx0pmlkj1j30uc06wq6x.jpg)

FileSystemXmlApplicationContext.java最终调用的构造器：



```
public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
      throws BeansException {

//为了动态的确定用哪个加载器去加载我们的配置文件
   super(parent);
   //告诉读取器     配置文件放在哪里  ：定位， 为了加载配置文件 
   setConfigLocations(configLocations);
   if (refresh) {
      refresh();
   }
}
```

```java

//解析bean定义资源文件的路径， 处理多个资源文件字符串数组
public void setConfigLocations(String... locations) {
   if (locations != null) {
      Assert.noNullElements(locations, "Config locations must not be null");
      this.configLocations = new String[locations.length];
      for (int i = 0; i < locations.length; i++) {
          //resolvePath为同一个类中将字符串解析为路径的方法。。
         this.configLocations[i] = resolvePath(locations[i]).trim();
      }
   }
   else {
      this.configLocations = null;
   }
}
```

![image-20190106180750425](https://ws3.sinaimg.cn/large/006tNc79ly1fyx0xl4jm0j310z0ikh35.jpg)

![image-20190106183518233](https://ws4.sinaimg.cn/large/006tNc79ly1fyx1q5sqifj312z0eyk5z.jpg)

![image-20190106184547552](https://ws2.sinaimg.cn/large/006tNc79ly1fyx212nyylj312m07vagq.jpg)

解析Bean定义资源文件的路径， 处理多个资源文件字符串数组：

![image-20190106184823577](https://ws4.sinaimg.cn/large/006tNc79ly1fyx23s39iaj312t0j2amv.jpg)

![image-20190106184953170](https://ws1.sinaimg.cn/large/006tNc79ly1fyx25cat9vj313a0iynkm.jpg)











![image-20190107010016226](https://ws1.sinaimg.cn/large/006tNc79ly1fyxcuq8a14j313i0kx1kx.jpg)