# spring源码解读一：IOC容器

[TOC]

## 1. IOC接口设计

![DefaultListableBeanFactory](https://ws2.sinaimg.cn/large/006tNc79ly1fywy75jejtj315a0psjsl.jpg)

>  其中BeanFactory作为最顶层的一个接口类，它定义了IOC容器的基本功能规范，BeanFactory 有三个子类：ListableBeanFactory、HierarchicalBeanFactory 和AutowireCapableBeanFactory。但是从上图中我们可以发现最终的默认实现类是 DefaultListableBeanFactory，他实现了所有的接口。那为何要定义这么多层次的接口呢？查阅这些接口的源码和说明发现，每个接口都有他使用的场合，它主要是为了区分在 Spring 内部在操作过程中对象的传递和转化过程中，对对象的数据访问所做的限制。例如 ListableBeanFactory 接口表示这些 Bean 是可列表的，而 HierarchicalBeanFactory 表示的是这些 Bean 是有继承关系的，也就是每个Bean 有可能有父 Bean。AutowireCapableBeanFactory 接口定义 Bean 的自动装配规则。这四个接口共同定义了 Bean 的集合、Bean 之间的关系、以及 Bean 行为。



## 2.BeanFactory接口

```java
public interface BeanFactory {

   /**
    * 对factoryBean的转义定义， 因为如果使用bean的名字检索FactoryBean得到的对象是工厂生成的对象，
    * 如果需要得到工厂本身， 需要转义
    */
   String FACTORY_BEAN_PREFIX = "&";
  //根据bean的名字，获取在ioc容器中得到的bean实例
   Object getBean(String name) throws BeansException;

  // 根据bean的名字和class类型来得到bean实例，增加了类型安全验证机制
   <T> T getBean(String name, Class<T> requiredType) throws BeansException;
    
   //提供对bean的检索， 看看是否ioc容器中是否有这个名字的bean
   boolean containsBean(String name);

   // 根据bean名字得到bean实例， 并同时判断这个bean是否是单例
   boolean isSingleton(String name) throws NoSuchBeanDefinitionException;

   //根据bean名字得到bean实例， 并同时判断这个bean是否是原型
   boolean isPrototype(String name) throws NoSuchBeanDefinitionException;

   //得到bean实例的class类型
   Class<?> getType(String name) throws NoSuchBeanDefinitionException;

   // 得到bean的别名，如果根据别名检索， 那么其原名也会被检索出来
   String[] getAliases(String name);

}
```

>  在BeanFactory里只对IOC容器的基本行为作了定义，根本不关心你的bean是如何定义怎样加载的。正如我们只关心工厂里得到什么的产品对象，至于工厂是怎么生产这些对象的，这个基本的接口不关心。

>   而要知道工厂是如何产生对象的，我们需要看具体的IOC容器实现，Spring提供许多IOC容器的实现。比如xmlBeanFactory, ClassPathXmlApplicationContext等。其中XmlBeanFactory就是针对最基本的Ioc容器的实现， 这个ioc容器可以读取xml文件定义的BeanDefinition(xml文件中对bean的描述)， 如果说xmlBeanfactory是容器中的屌丝， applicationContext应该是容器中的高富帅。

> applicationContext是spring提供的一个高级的ioc容器，它除了能够提供ioc容器的基本功能外， 还为用户提供了以下的服务。
>
> 从ApplicationContext接口的实现，我们看出其特点：
>
> 1.支持信息源， 可以实现国际化（实现MessageSource接口）
>
> 2.访问资源（实现ResourcePatternResolver接口）
>
> 3.支持应用事件。（实现ApplicationEventPublisher接口）

**具体的继承关系看下图：**

![image-20190106174841527](https://ws3.sinaimg.cn/large/006tNc79ly1fyx0dxldz9j31fc0pg77t.jpg)



## 3.BeanDefinition

spring Ioc容器管理我们定义的各种bean对象及其相互的关系，bean对象在spring实现中是以beanDefinition来描述的，其继承体系如下：

![image-20190106170501030](https://ws1.sinaimg.cn/large/006tNc79ly1fywz47tvp7j31fa0u00wf.jpg)

## 4.IOC容器的初始化

IOC容器的初始化包括BeanDefinition的Resource定位、载入和注册这三个基本的过程。

1.定位:定位资源（定位查找xml配置文件 得到一个Resource）

2.加载: 已经找到配置文件  

3.注册: 解析配置文件并封装成BeanDefinition(此时只是对bean的说明而已， bean还没有真正的产生)

## 5. IOC容器的源码解析

> XmlBeanFactory是IOC容器系列最底层的实现，它继承自DefaultListableBeanFactory这个类。而后者是Spring中非常重要的一个类，它是Spring容器中一个基本产品，可以把它当做一个默认的功能完整的IOC容器来使用。

> XmlBeanFactory除了从DefaultListableBeanFactory继承到IOC容器基本功能之外，还新增了一些其他功能，从名称就可以猜测出来，它是一个可以读取以XML文件方式定义BeanDefinition的容器。

**XmlBeanFactory应用**

```java
Resource rs=new FileSystemResource("applicationContext01.xml");
BeanFactory ct=new XmlBeanFactory(rs);
ct.getBean("");
```

**XmlBeanFactory源码如下：**

```java
public class XmlBeanFactory extends DefaultListableBeanFactory {

   private final XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(this);

   public XmlBeanFactory(Resource resource) throws BeansException {
      this(resource, null);
   }
    
   public XmlBeanFactory(Resource resource, BeanFactory parentBeanFactory) throws BeansException {
      super(parentBeanFactory);
      this.reader.loadBeanDefinitions(resource);
   }

}
```
实际上，实现XML读取功能并不是直接由XmlBeanFactory来完成的。而是由XmlBeanFactory内部定义的XmlBeanDefinitionReader来进行处理的。在构造XmlBeanFactory容器的时候，需要给出BeanDefinition的信息来源，而这个信息来源需要封装成Spring中的Resource类的形式给出。

来看下一个基本的IOC容器的初始化过程：

1、创建IOC配置文件的抽象资源，也就是源码中的Resource，这个Resource中包含了BeanDefinition的定义信息。

2、通过构造函数创建一个BeanFactory。

3、创建一个载入BeanDefinition的读取器，即源码中的reader。这里使用XmlBeanDefinitionReader来载入XML文件形式的BeanDefinition。

4、调用reader的loadBeanDefinitions方法，来完成从Resource中载入BeanDefinition信息，从而完成IOC容器的初始化。

我们可以将上面的源码做简化，使用编程式的方式来表达IOC容器的初始化：

```JAVA
//根据Xml配置文件创建Resource资源对象，该对象中包含了BeanDefinition的信息
 ClassPathResource resource =new ClassPathResource("application-context.xml");
//创建DefaultListableBeanFactory
 DefaultListableBeanFactory factory =new DefaultListableBeanFactory();
//创建XmlBeanDefinitionReader读取器，用于载入BeanDefinition。之所以需要BeanFactory作为参数，是因为会将读取的信息回调配置给factory
 XmlBeanDefinitionReader reader =new XmlBeanDefinitionReader(factory);
//XmlBeanDefinitionReader执行载入BeanDefinition的方法，最后会完成Bean的载入和注册。完成后Bean就成功的放置到IOC容器当中，以后我们就可以从中取得Bean来使用
 reader.loadBeanDefinitions(resource);
```



**高富帅IOC解析：这里我们以FileSystemXmlApplicationContext为例**

```java
ApplicationContext ct=new FileSystemXmlApplicationContext("applicationContext01.xml");
ct.getBean("");
```



**FileSystemXmlApplicationContext.java最终调用的构造器**

```java
public FileSystemXmlApplicationContext(String[] configLocations, boolean refresh, ApplicationContext parent)
      throws BeansException {

//调用父类容器的构造方法为容器设置好Bean资源加载器
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





### refresh()方法

> **refresh函数在AbstractApplicationContext类中;Spring类中  IoC容器对Bean定义资源的载入是从refresh()函数开始的，refresh()是一个模板方法，refresh()方法的作用是：在创建IoC容器前，如果已经有容器存在，则需要把已有的容器销毁和关闭，以保证在refresh之后使用的是新建立起来的IoC容器。refresh的作用类似于对IoC容器的重启，在新建立好的容器中对容器进行初始化，对Bean定义资源进行载入，FileSystemXmlApplicationContext通过调用其父类AbstractApplicationContext的refresh()函数启动整个IoC容器对Bean定义的载入过程**

```java
//AbstractApplicationContext.java

public void refresh() throws BeansException, IllegalStateException {  
       synchronized (this.startupShutdownMonitor) {  
           //调用容器准备刷新的方法，获取容器的当时时间，同时给容器设置同步标识  
           prepareRefresh();  
           //告诉子类启动refreshBeanFactory()方法，Bean定义资源文件的载入从  
          //子类的refreshBeanFactory()方法启动  
           ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();  
           //为BeanFactory配置容器特性，例如类加载器、事件处理器等  
           prepareBeanFactory(beanFactory);  
           try {  
               //为容器的某些子类指定特殊的BeanPost事件处理器  
               postProcessBeanFactory(beanFactory);  
               //调用所有注册的BeanFactoryPostProcessor的Bean  
               invokeBeanFactoryPostProcessors(beanFactory);  
               //为BeanFactory注册BeanPost事件处理器.  
               //BeanPostProcessor是Bean后置处理器，用于监听容器触发的事件  
               registerBeanPostProcessors(beanFactory);  
               //初始化信息源，和国际化相关.  
               initMessageSource();  
               //初始化容器事件传播器.  
               initApplicationEventMulticaster();  
               //调用子类的某些特殊Bean初始化方法  
               onRefresh();  
               //为事件传播器注册事件监听器.  
               registerListeners();  
               //初始化所有剩余的单态Bean.  
               finishBeanFactoryInitialization(beanFactory);  
               //初始化容器的生命周期事件处理器，并发布容器的生命周期事件  
               finishRefresh();  
           }  
           catch (BeansException ex) {  
               //销毁以创建的单态Bean  
               destroyBeans();  
               //取消refresh操作，重置容器的同步标识.  
               cancelRefresh(ex);  
               throw ex;  
           }  
       }  
   }
```

>  refresh()方法主要为IoC容器Bean的生命周期管理提供条件，Spring IoC容器载入Bean定义资源文件从其子类容器的refreshBeanFactory()方法启动，所以整个refresh()中“ConfigurableListableBeanFactory beanFactory =obtainFreshBeanFactory();”这句以后代码的都是注册容器的信息源和生命周期事件，载入过程就是从这句代码启动。

> refresh()方法的作用是：在创建IoC容器前，如果已经有容器存在，则需要把已有的容器销毁和关闭，以保证在refresh之后使用的是新建立起来的IoC容器。refresh的作用类似于对IoC容器的重启，在新建立好的容器中对容器进行初始化，对Bean定义资源进行载入



### obtainFreshBeanFactory()方法

> AbstractApplicationContext的obtainFreshBeanFactory()方法调用子类容器的refreshBeanFactory()方法，启动容器载入Bean定义资源文件的过程

```java
//AbstractApplicationContext.java

protected ConfigurableListableBeanFactory obtainFreshBeanFactory() {  
        //这里使用了委派设计模式，父类定义了抽象的refreshBeanFactory()方法，具体实现调用子类容器的refreshBeanFactory()方法
         refreshBeanFactory();  
        ConfigurableListableBeanFactory beanFactory = getBeanFactory();  
        if (logger.isDebugEnabled()) {  
            logger.debug("Bean factory for " + getDisplayName() + ": " + beanFactory);  
        }  
        return beanFactory;  
    }
```

### refreshBeanFactory()方法

>  AbstractApplicationContext类中只抽象定义了refreshBeanFactory()方法，容器真正调用的是其子类AbstractRefreshableApplicationContext实现的 refreshBeanFactory()方法，

```java
//AbstractRefreshableApplicationContext.java

protected final void refreshBeanFactory() throws BeansException {  
       if (hasBeanFactory()) {//如果已经有容器，销毁容器中的bean，关闭容器  
           destroyBeans();  
           closeBeanFactory();  
       }  
       try {  
            //创建IoC容器  
            DefaultListableBeanFactory beanFactory = createBeanFactory();  
            beanFactory.setSerializationId(getId());  
           //对IoC容器进行定制化，如设置启动参数，开启注解的自动装配等  
           customizeBeanFactory(beanFactory);  
           //调用载入Bean定义的方法，主要这里又使用了一个委派模式，在当前类中只定义了抽象的loadBeanDefinitions方法，具体的实现调用子类容器  
           loadBeanDefinitions(beanFactory);  
           synchronized (this.beanFactoryMonitor) {  
               this.beanFactory = beanFactory;  
           }  
       }  
       catch (IOException ex) {  
           throw new ApplicationContextException("I/O error parsing bean definition source for " + getDisplayName(), ex);  
       }  
   }
```

> 在这个方法中，先判断BeanFactory是否存在，如果存在则先销毁beans并关闭beanFactory，接着创建DefaultListableBeanFactory，并调用loadBeanDefinitions(beanFactory)装载bean定义

### loadBeanDefinitions()

> loadBeanDefinitions()方法同样是抽象方法，是由其子类实现的，也即在AbstractXmlApplicationContext中

```java
//AbstractXmlApplicationContext.java

public abstract class AbstractXmlApplicationContext extends AbstractRefreshableConfigApplicationContext {  
    ……  
    //实现父类抽象的载入Bean定义方法  
    @Override  
    protected void loadBeanDefinitions(DefaultListableBeanFactory beanFactory) throws BeansException, IOException {  
        //创建XmlBeanDefinitionReader，即创建Bean读取器，并通过回调设置到容器中去，容  器使用该读取器读取Bean定义资源  
        XmlBeanDefinitionReader beanDefinitionReader = new XmlBeanDefinitionReader(beanFactory);  
        //为Bean读取器设置Spring资源加载器，AbstractXmlApplicationContext的  
        //祖先父类AbstractApplicationContext继承DefaultResourceLoader，因此，容器本身也是一个资源加载器  
       beanDefinitionReader.setResourceLoader(this);  
       //为Bean读取器设置SAX xml解析器  
       beanDefinitionReader.setEntityResolver(new ResourceEntityResolver(this));  
       //当Bean读取器读取Bean定义的Xml资源文件时，启用Xml的校验机制  
       initBeanDefinitionReader(beanDefinitionReader);  
       //Bean读取器真正实现加载的方法  
       loadBeanDefinitions(beanDefinitionReader);  
   }  
    
   //Xml Bean读取器加载Bean定义资源  
   protected void loadBeanDefinitions(XmlBeanDefinitionReader reader) throws BeansException, IOException {  
       //获取Bean定义资源的定位  
       Resource[] configResources = getConfigResources();  
       if (configResources != null) {  
           //Xml Bean读取器调用其父类AbstractBeanDefinitionReader读取定位  
           //的Bean定义资源  
           reader.loadBeanDefinitions(configResources);  
       }  
       //如果子类中获取的Bean定义资源定位为空，则获取FileSystemXmlApplicationContext构造方法中setConfigLocations方法设置的资源  
       String[] configLocations = getConfigLocations();  
       if (configLocations != null) {  
           //Xml Bean读取器调用其父类AbstractBeanDefinitionReader读取定位  
           //的Bean定义资源  
           reader.loadBeanDefinitions(configLocations);  
       }  
   }  
   //这里又使用了一个委托模式，调用子类的获取Bean定义资源定位的方法  
   //该方法在ClassPathXmlApplicationContext中进行实现，对于我们  
   //举例分析源码的FileSystemXmlApplicationContext没有使用该方法  
   protected Resource[] getConfigResources() {  
       return null;  
   }   ……  
41}
```

> Xml Bean读取器(XmlBeanDefinitionReader)调用其父类AbstractBeanDefinitionReader的 reader.loadBeanDefinitions方法读取Bean定义资源。
>
> 由于我们使用FileSystemXmlApplicationContext作为例子分析，因此getConfigResources的返回值为null，因此程序执行reader.loadBeanDefinitions(configLocations)分支

### loadBeanDefinitions()方法

> **此时的loadBeanDefinitions发生在AbstractBeanDefinitionReader类中**

```java
//AbstractBeanDefinitionReader.java

//重载方法，调用下面的loadBeanDefinitions(String, Set<Resource>);方法  
   public int loadBeanDefinitions(String location) throws BeanDefinitionStoreException {  
       return loadBeanDefinitions(location, null);  
   }  
   public int loadBeanDefinitions(String location, Set<Resource> actualResources) throws BeanDefinitionStoreException {  
       //获取在IoC容器初始化过程中设置的资源加载器  
       ResourceLoader resourceLoader = getResourceLoader();  
       if (resourceLoader == null) {  
           throw new BeanDefinitionStoreException(  
                   "Cannot import bean definitions from location [" + location + "]: no ResourceLoader available");  
       }  
       if (resourceLoader instanceof ResourcePatternResolver) {  
           try {  
               //将指定位置的Bean定义资源文件解析为Spring IoC容器封装的资源  
               //加载多个指定位置的Bean定义资源文件  
               Resource[] resources = ((ResourcePatternResolver) resourceLoader).getResources(location);  
               //委派调用其子类XmlBeanDefinitionReader的方法，实现加载功能  
               int loadCount = loadBeanDefinitions(resources);  
               if (actualResources != null) {  
                   for (Resource resource : resources) {  
                       actualResources.add(resource);  
                   }  
               }  
               if (logger.isDebugEnabled()) {  
                   logger.debug("Loaded " + loadCount + " bean definitions from location pattern [" + location + "]");  
               }  
               return loadCount;  
           }  
           catch (IOException ex) {  
               throw new BeanDefinitionStoreException(  
                       "Could not resolve bean definition resource pattern [" + location + "]", ex);  
           }  
       }  
       else {  
           //将指定位置的Bean定义资源文件解析为Spring IoC容器封装的资源  
           //加载单个指定位置的Bean定义资源文件  
           Resource resource = resourceLoader.getResource(location);  
           //委派调用其子类XmlBeanDefinitionReader的方法，实现加载功能  
           int loadCount = loadBeanDefinitions(resource);  
           if (actualResources != null) {  
               actualResources.add(resource);  
           }  
           if (logger.isDebugEnabled()) {  
               logger.debug("Loaded " + loadCount + " bean definitions from location [" + location + "]");  
           }  
           return loadCount;  
       }  
   }  
   //重载方法，调用loadBeanDefinitions(String);  
   public int loadBeanDefinitions(String... locations) throws BeanDefinitionStoreException {  
       Assert.notNull(locations, "Location array must not be null");  
       int counter = 0;  
       for (String location : locations) {  
           counter += loadBeanDefinitions(location);  
       }  
       return counter;  
    }
```

> 从对AbstractBeanDefinitionReader的loadBeanDefinitions方法源码分析可以看出该方法做了以下两件事：
>  首先，调用资源加载器的获取资源方法resourceLoader.getResource(location)，获取到要加载的资源。
>  其次，真正执行加载功能是其子类XmlBeanDefinitionReader的loadBeanDefinitions方法





### doLoadDocument()方法

> **载入Bean定义资源文件的最后一步是将Bean定义资源转换为Document对象，该过程由doLoadDocument实现，dOLoadDocument方法在XmlBeanDefinitionReader中**

```java
//XmlBeanDefinitionReader.java

protected Document doLoadDocument(InputSource inputSource, Resource resource) throws Exception {
		return this.documentLoader.loadDocument(inputSource, getEntityResolver(), this.errorHandler,
				getValidationModeForResource(resource), isNamespaceAware());
	}


---------------------------------------------------------------------------------------
 //该方法发生在DefaultDocumentLoader.java中
    
//使用标准的JAXP将载入的Bean定义资源转换成document对象  
   public Document loadDocument(InputSource inputSource, EntityResolver entityResolver,  
           ErrorHandler errorHandler, int validationMode, boolean namespaceAware) throws Exception {  
       //创建文件解析器工厂  
       DocumentBuilderFactory factory = createDocumentBuilderFactory(validationMode, namespaceAware);  
       if (logger.isDebugEnabled()) {  
           logger.debug("Using JAXP provider [" + factory.getClass().getName() + "]");  
       }  
       //创建文档解析器  
       DocumentBuilder builder = createDocumentBuilder(factory, entityResolver, errorHandler);  
       //解析Spring的Bean定义资源  
       return builder.parse(inputSource);  
   }  


   protected DocumentBuilderFactory createDocumentBuilderFactory(int validationMode, boolean namespaceAware)  
           throws ParserConfigurationException {  
       //创建文档解析工厂  
       DocumentBuilderFactory factory = DocumentBuilderFactory.newInstance();  
       factory.setNamespaceAware(namespaceAware);  
       //设置解析XML的校验  
       if (validationMode != XmlValidationModeDetector.VALIDATION_NONE) {  
           factory.setValidating(true);  
           if (validationMode == XmlValidationModeDetector.VALIDATION_XSD) {  
               factory.setNamespaceAware(true);  
               try {  
                   factory.setAttribute(SCHEMA_LANGUAGE_ATTRIBUTE, XSD_SCHEMA_LANGUAGE);  
               }  
               catch (IllegalArgumentException ex) {  
                   ParserConfigurationException pcex = new ParserConfigurationException(  
                           "Unable to validate using XSD: Your JAXP provider [" + factory +  
                           "] does not support XML Schema. Are you running on Java 1.4 with Apache Crimson? " +  
                           "Upgrade to Apache Xerces (or Java 1.5) for full XSD support.");  
                   pcex.initCause(ex);  
                   throw pcex;  
               }  
           }  
       }  
       return factory;  
   }
```

> 至此Spring IoC容器根据定位的Bean定义资源文件，将其加载读入并转换成为Document对象过程完成。接下来我们要继续分析Spring IoC容器将载入的Bean定义资源文件转换为Document对象之后，是如何将其解析为Spring IoC管理的Bean对象并将其注册到容器中的。



### registerBeanDefinitions方法

> **registerBeanDefinitions方法发生在XmlBeanDefinitionReader方法中**

```java
//XmlBeanDefinitionReader.java

//按照Spring的Bean语义要求将Bean定义资源解析并转换为容器内部数据结构  
   public int registerBeanDefinitions(Document doc, Resource resource) throws BeanDefinitionStoreException {  
       //得到BeanDefinitionDocumentReader来对xml格式的BeanDefinition解析  
       BeanDefinitionDocumentReader documentReader = createBeanDefinitionDocumentReader();  
       //获得容器中注册的Bean数量  
       int countBefore = getRegistry().getBeanDefinitionCount();  
       //解析过程入口，这里使用了委派模式，BeanDefinitionDocumentReader只是个接口，//具体的解析实现过程有实现类DefaultBeanDefinitionDocumentReader完成  
       documentReader.registerBeanDefinitions(doc, createReaderContext(resource));  
       //统计解析的Bean数量  
       return getRegistry().getBeanDefinitionCount() - countBefore;  
   }  


   //创建BeanDefinitionDocumentReader对象，解析Document对象  
   protected BeanDefinitionDocumentReader createBeanDefinitionDocumentReader() {  
       return BeanDefinitionDocumentReader.class.cast(BeanUtils.instantiateClass(this.documentReaderClass));  
      }

---------------------------------------------------------------------------------------
 //DefaultBeanDefinitionDocumentReader.java   

public void registerBeanDefinitions(Document doc, XmlReaderContext readerContext) {
		this.readerContext = readerContext;
		logger.debug("Loading bean definitions");
		Element root = doc.getDocumentElement();
		doRegisterBeanDefinitions(root);
	}

protected void doRegisterBeanDefinitions(Element root) {
		BeanDefinitionParserDelegate parent = this.delegate;
		this.delegate = createDelegate(getReaderContext(), root, parent);

		if (this.delegate.isDefaultNamespace(root)) {
			String profileSpec = root.getAttribute(PROFILE_ATTRIBUTE);
			if (StringUtils.hasText(profileSpec)) {
				String[] specifiedProfiles = StringUtils.tokenizeToStringArray(
						profileSpec, BeanDefinitionParserDelegate.MULTI_VALUE_ATTRIBUTE_DELIMITERS);
				if (!getReaderContext().getEnvironment().acceptsProfiles(specifiedProfiles)) {
					return;
				}
			}
		}

     //在解析Bean定义之前，进行自定义的解析，增强解析过程的可扩展性  
		preProcessXml(root);
     //从Document的根元素开始进行Bean定义的Document对象 (**重点**)
		parseBeanDefinitions(root, this.delegate);
     //在解析Bean定义之后，进行自定义的解析，增加解析过程的可扩展性  
		postProcessXml(root);

		this.delegate = parent;
	}

```

> Bean定义资源的载入解析分为以下两个过程：
>
> 首先，通过调用XML解析器将Bean定义资源文件转换得到Document对象，但是这些Document对象并没有按照Spring的Bean规则进行解析。这一步是载入的过程
>
> 其次，在完成通用的XML解析之后，按照Spring的Bean规则对Document对象进行解析。
>
> 按照Spring的Bean规则对Document对象解析的过程是在接口BeanDefinitionDocumentReader的实现类DefaultBeanDefinitionDocumentReader中实现的。BeanDefinitionDocumentReader接口通过registerBeanDefinitions方法调用其实现类DefaultBeanDefinitionDocumentReader对Document对象进行解析

### parseBeanDefinitions方法

> **parseBeanDefinitions方法发生在DefaultBeanDefinitionDocumentReader中**

```java
//DefaultBeanDefinitionDocumentReader.java

protected void parseBeanDefinitions(Element root, BeanDefinitionParserDelegate delegate) {
   if (delegate.isDefaultNamespace(root)) {
      NodeList nl = root.getChildNodes();
      for (int i = 0; i < nl.getLength(); i++) {
         Node node = nl.item(i);
         if (node instanceof Element) {
            Element ele = (Element) node;
             //Bean定义的Document的元素节点使用的是Spring默认的XML命名空间 
            if (delegate.isDefaultNamespace(ele)) {
               parseDefaultElement(ele, delegate);
            }
            else {
               delegate.parseCustomElement(ele);
            }
         }
      }
   }
   else {
      delegate.parseCustomElement(root);
   }
}


 //使用Spring的Bean规则解析Document元素节点 
	private void parseDefaultElement(Element ele, BeanDefinitionParserDelegate delegate) {
         //如果元素节点是<Import>导入元素，进行导入解析  
		if (delegate.nodeNameEquals(ele, IMPORT_ELEMENT)) {
			importBeanDefinitionResource(ele);
		}
        //如果元素节点是<Alias>别名元素，进行别名解析 
		else if (delegate.nodeNameEquals(ele, ALIAS_ELEMENT)) {
			processAliasRegistration(ele);
		}
         //元素节点既不是导入元素，也不是别名元素，即普通的<Bean>元素，
		else if (delegate.nodeNameEquals(ele, BEAN_ELEMENT)) {
			processBeanDefinition(ele, delegate);
		}
		else if (delegate.nodeNameEquals(ele, NESTED_BEANS_ELEMENT)) {
			// recurse
			doRegisterBeanDefinitions(ele);
		}
	}

//解析Bean定义资源Document对象的普通元素  
172    protected void processBeanDefinition(Element ele, BeanDefinitionParserDelegate delegate) {  
173        // BeanDefinitionHolder是对BeanDefinition的封装，即Bean定义的封装类  
174        //对Document对象中<Bean>元素的解析由BeanDefinitionParserDelegate实现  
    BeanDefinitionHolder bdHolder = delegate.parseBeanDefinitionElement(ele);  
175        if (bdHolder != null) {  
176            bdHolder = delegate.decorateBeanDefinitionIfRequired(ele, bdHolder);  
177            try {  
178               //向Spring IoC容器注册解析得到的Bean定义，这是Bean定义向IoC容器注册的入口            
                  BeanDefinitionReaderUtils.registerBeanDefinition(bdHolder, getReaderContext().getRegistry());  
179            }  
180            catch (BeanDefinitionStoreException ex) {  
181                getReaderContext().error("Failed to register bean definition with name '" +  
182                        bdHolder.getBeanName() + "'", ele, ex);  
183            }  
184            //在完成向Spring IoC容器注册解析得到的Bean定义之后，发送注册事件  
185            getReaderContext().fireComponentRegistered(new BeanComponentDefinition(bdHolder));  
186        }  
187    } 

```



###registerBeanDefinition方法

>  **registerBeanDefinition方法发生在DefaultListableBeanFactory中**

```java
@Override
public void registerBeanDefinition(String beanName, BeanDefinition beanDefinition)
      throws BeanDefinitionStoreException {

   Assert.hasText(beanName, "Bean name must not be empty");
   Assert.notNull(beanDefinition, "BeanDefinition must not be null");

   if (beanDefinition instanceof AbstractBeanDefinition) {
      try {
         ((AbstractBeanDefinition) beanDefinition).validate();
      }
      catch (BeanDefinitionValidationException ex) {
         throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
               "Validation of bean definition failed", ex);
      }
   }

   BeanDefinition oldBeanDefinition;

   oldBeanDefinition = this.beanDefinitionMap.get(beanName);
   if (oldBeanDefinition != null) {
      if (!isAllowBeanDefinitionOverriding()) {
         throw new BeanDefinitionStoreException(beanDefinition.getResourceDescription(), beanName,
               "Cannot register bean definition [" + beanDefinition + "] for bean '" + beanName +
               "': There is already [" + oldBeanDefinition + "] bound.");
      }
      else if (oldBeanDefinition.getRole() < beanDefinition.getRole()) {
         // e.g. was ROLE_APPLICATION, now overriding with ROLE_SUPPORT or ROLE_INFRASTRUCTURE
         if (this.logger.isWarnEnabled()) {
            this.logger.warn("Overriding user-defined bean definition for bean '" + beanName +
                  "' with a framework-generated bean definition: replacing [" +
                  oldBeanDefinition + "] with [" + beanDefinition + "]");
         }
      }
      else if (!beanDefinition.equals(oldBeanDefinition)) {
         if (this.logger.isInfoEnabled()) {
            this.logger.info("Overriding bean definition for bean '" + beanName +
                  "' with a different definition: replacing [" + oldBeanDefinition +
                  "] with [" + beanDefinition + "]");
         }
      }
      else {
         if (this.logger.isDebugEnabled()) {
            this.logger.debug("Overriding bean definition for bean '" + beanName +
                  "' with an equivalent definition: replacing [" + oldBeanDefinition +
                  "] with [" + beanDefinition + "]");
         }
      }
       //注册
      this.beanDefinitionMap.put(beanName, beanDefinition);
   }
   else {
      if (hasBeanCreationStarted()) {
         // Cannot modify startup-time collection elements anymore (for stable iteration)
         synchronized (this.beanDefinitionMap) {
            this.beanDefinitionMap.put(beanName, beanDefinition);
            List<String> updatedDefinitions = new ArrayList<String>(this.beanDefinitionNames.size() + 1);
            updatedDefinitions.addAll(this.beanDefinitionNames);
            updatedDefinitions.add(beanName);
            this.beanDefinitionNames = updatedDefinitions;
            if (this.manualSingletonNames.contains(beanName)) {
               Set<String> updatedSingletons = new LinkedHashSet<String>(this.manualSingletonNames);
               updatedSingletons.remove(beanName);
               this.manualSingletonNames = updatedSingletons;
            }
         }
      }
      else {
         // Still in startup registration phase
         this.beanDefinitionMap.put(beanName, beanDefinition);
         this.beanDefinitionNames.add(beanName);
         this.manualSingletonNames.remove(beanName);
      }
      this.frozenBeanDefinitionNames = null;
   }

   if (oldBeanDefinition != null || containsSingleton(beanName)) {
      resetBeanDefinition(beanName);
   }
}
```



## 总结

​    **现在通过上面的代码，总结一下IOC容器初始化的基本步骤：**

-  初始化的入口在容器实现中的 refresh()调用来完成

- 对 bean 定义载入 IOC 容器使用的方法是 loadBeanDefinition,其中的大致过程如下：通过 ResourceLoader 来完成资源文件位置的定位，DefaultResourceLoader 是默认的实现，同时上下文本身就给出了 ResourceLoader 的实现，可以从类路径，文件系统, URL 等方式来定为资源位置。如果是 XmlBeanFactory作为 IOC 容器，那么需要为它指定 bean 定义的资源，也就是说 bean 定义文件时通过抽象成 Resource 来被 IOC 容器处理的，容器通过 BeanDefinitionReader来完成定义信息的解析和 Bean 信息的注册,往往使用的是XmlBeanDefinitionReader 来解析 bean 的 xml 定义文件 - 实际的处理过程是委托给 BeanDefinitionParserDelegate 来完成的，从而得到 bean 的定义信息，这些信息在 Spring 中使用 BeanDefinition 对象来表示 - 这个名字可以让我们想到loadBeanDefinition,RegisterBeanDefinition  这些相关的方法 - 他们都是为处理 BeanDefinitin 服务的， 容器解析得到 BeanDefinitionIoC 以后，需要把它在 IOC 容器中注册，这由 IOC 实现 BeanDefinitionRegistry 接口来实现。注册过程就是在 IOC 容器内部维护的一个HashMap 来保存得到的 BeanDefinition 的过程。这个 HashMap 是 IoC 容器持有 bean 信息的场所，以后对 bean 的操作都是围绕这个HashMap 来实现的.

-  然后我们就可以通过 BeanFactory 和 ApplicationContext 来享受到 Spring IOC 的服务了,在使用 IOC 容器的时候，我们注意到除了少量粘合代码，绝大多数以正确 IoC 风格编写的应用程序代码完全不用关心如何到达工厂，因为容器将把这些对象与容器管理的其他对象钩在一起。基本的策略是把工厂放到已知的地方，最好是放在对预期使用的上下文有意义的地方，以及代码将实际需要访问工厂的地方。 Spring 本身提供了对声明式载入 web 应用程序用法的应用程序上下文,并将其存储在ServletContext 中的框架实现。具体可以参见以后的文章 

- 在使用 Spring IOC 容器的时候我们还需要区别两个概念:

 - - Beanfactory 和 Factory bean，其中 BeanFactory 指的是 IOC 容器的编程抽象，比如 ApplicationContext， XmlBeanFactory 等，这些都是 IOC 容器的具体表现，需要使用什么样的容器由客户决定,但 Spring 为我们提供了丰富的选择。 FactoryBean 只是一个可以在 IOC而容器中被管理的一个 bean,是对各种处理过程和资源使用的抽象,Factory bean 在需要时产生另一个对象，而不返回 FactoryBean本身,我们可以把它看成是一个抽象工厂，对它的调用返回的是工厂生产的产品。所有的 Factory bean 都实现特殊的org.springframework.beans.factory.FactoryBean 接口，当使用容器中 factory bean 的时候，该容器不会返回 factory bean 本身,而是返回其生成的对象。Spring 包括了大部分的通用资源和服务访问抽象的 Factory bean 的实现，其中包括:对 JNDI 查询的处理，对代理对象的处理，对事务性代理的处理，对 RMI 代理的处理等，这些我们都可以看成是具体的工厂,看成是SPRING 为我们建立好的工厂。也就是说 Spring 通过使用抽象工厂模式为我们准备了一系列工厂来生产一些特定的对象,免除我们手工重复的工作，我们要使用时只需要在 IOC 容器里配置好就能很方便的使用了
