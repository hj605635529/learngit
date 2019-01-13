# Spring源码解读二：依赖注入实现原理

[TOC]

## 1.依赖注入介绍

在分析原理之前我们先回顾下依赖注入的概念：

>我们常提起的依赖注入(Dependency Injection)和控制反转(Inversion of Control)是同一个概念。具体含义是:当某个角色(可能是一个Java实例，调用者)需要另一个角色(另一个Java实例，被调用者)的协助时，在 传统的程序设计过程中，通常由调用者来创建被调用者的实例。但在Spring里，创建被调用者的工作不再由调用者来完成，因此称为控制反转;创建被调用者 实例的工作通常由Spring容器来完成，然后注入调用者，因此也称为依赖注入。

## 2.依赖注入发生的时间

当Spring IOC容器完成了Bean定义资源的定位、载入和解析注册以后， IOC容器中已经管理类Bean定义的相关数据，但是此时Ioc容器还没有对所管理的Bean进行依赖注入，依赖注入在以下两种情况发生：

- 用户第一次通过getBean方法向IOC容器索要Bean时，IOC容器发生依赖注入。
- 当用户在Bean定义资源中为<Bean>元素配置了lazy-init属性，既让容器在解析注册Bean定义时进行预实例化，触发依赖注入。

>  BeanFactory接口定义了Spring IOC容器的基本功能规范，是springIoc容器所应该遵守的最底层和最基本的编码规范。BeanFactory接口中定义了几个getBean方法，就是用户向Ioc容器索取管理的Bean的方法。我们通过分析其子类的具体实现，理解Spring IOC容器在用户索取Bean时如何完成依赖注入。

## 3. 源码剖析

之前的文章分析到了调用getBean获得Bean对象之前的步骤，而我们调用getBean后获得已经是完成依赖关系注入的完整Bean对象，所以完成依赖注入的奥秘就发生在getBean当中，要分析我们便从getBean这个入口开始吧。

### 3.1 继承体系

![image-20190113231513612](https://ws4.sinaimg.cn/large/006tNc79ly1fz5d5kxxhkj31mj0u0td3.jpg)



### 3.2 getBean()方法源码

> **先看看getBean()方法的源码， 方法的实现在AbstractBeanFactory.java中**

```java
//AbstractBeanFactory.java

//---------------------------------------------------------------------
// Implementation of BeanFactory interface
//---------------------------------------------------------------------

//获取IOC容器中指定名称的Bean 
@Override
public Object getBean(String name) throws BeansException {
    //doGetBean才是真正向IoC容器获取被管理Bean的过程
   return doGetBean(name, null, null, false);
}

//获取IOC容器中指定名称和类型的Bean
@Override
public <T> T getBean(String name, Class<T> requiredType) throws BeansException {
   return doGetBean(name, requiredType, null, false);
}

//获取IOC容器中指定名称和参数的Bean
@Override
public Object getBean(String name, Object... args) throws BeansException {
   return doGetBean(name, null, args, false);
}

//获取IOC容器中指定名称、类型和参数的Bean
public <T> T getBean(String name, Class<T> requiredType, Object... args) throws BeansException {
   return doGetBean(name, requiredType, args, false);
}
```

几个重载的getBean方法的实现都调用了 **doGetBean** 这个方法，继续看 `doGetBean()`

```java
//AbstractBeanFactory.java

//真正实现向IOC容器获取Bean的功能，也是触发依赖注入功能的地方
@SuppressWarnings("unchecked")
protected <T> T doGetBean(
      final String name, final Class<T> requiredType, final Object[] args, boolean typeCheckOnly)
      throws BeansException {

   //1.根据指定的名称获取被管理Bean的名称，剥离指定名称中对容器的相关依赖
    //如果指定的是别名，将别名转换为规范的Bean名称
   final String beanName = transformedBeanName(name);
   Object bean;

   //2.先从缓存中取是否已经有被创建过的单态类型的Bean
    //对于单例模式的Bean整个IOC容器中只创建一次，不需要重复创建
   Object sharedInstance = getSingleton(beanName);
    //IOC容器创建单例模式Bean实例对象
   if (sharedInstance != null && args == null) {
      if (logger.isDebugEnabled()) {
         //如果指定名称的Bean在容器中已有单例模式的Bean被创建
            //直接返回已经创建的Bean
         if (isSingletonCurrentlyInCreation(beanName)) {
            logger.debug("Returning eagerly cached instance of singleton bean '" + beanName +
                  "' that is not fully initialized yet - a consequence of a circular reference");
         }
         else {
            logger.debug("Returning cached instance of singleton bean '" + beanName + "'");
         }
      }
          //获取给定Bean的实例对象，主要是完成FactoryBean的相关处理  
           //注意：BeanFactory是管理容器中Bean的工厂，而FactoryBean是  
           //创建创建对象的工厂Bean，两者之间有区别
      bean = getObjectForBeanInstance(sharedInstance, name, beanName, null);
   }

   else {
      //缓存没有正在创建的单例模式Bean  
        //缓存中已经有已经创建的原型模式Bean
        //但是由于循环引用的问题导致实例化对象失败
      if (isPrototypeCurrentlyInCreation(beanName)) {
         throw new BeanCurrentlyInCreationException(beanName);
      }

      // Check if bean definition exists in this factory.
      //对IOC容器中是否存在指定名称的BeanDefinition进行检查，首先检查是否  
        //能在当前的BeanFactory中获取的所需要的Bean，如果不能则委托当前容器  
        //的父级容器去查找，如果还是找不到则沿着容器的继承体系向父级容器查找
      BeanFactory parentBeanFactory = getParentBeanFactory();
      //当前容器的父级容器存在，且当前容器中不存在指定名称的Bean
      if (parentBeanFactory != null && !containsBeanDefinition(beanName)) {
         // Not found -> check parent.
         //解析指定Bean名称的原始名称
         String nameToLookup = originalBeanName(name);
         if (args != null) {
            // Delegation to parent with explicit args.
            //委派父级容器根据指定名称和显式的参数查找
            return (T) parentBeanFactory.getBean(nameToLookup, args);
         }
         else {
            // No args -> delegate to standard getBean method.
            //委派父级容器根据指定名称和类型查找
            return parentBeanFactory.getBean(nameToLookup, requiredType);
         }
      }

      //创建的Bean是否需要进行类型验证，一般不需要
      if (!typeCheckOnly) {
         //向容器标记指定的Bean已经被创建
         markBeanAsCreated(beanName);
      }

      try {
         //根据指定Bean名称获取其父级的Bean定义
           //主要解决Bean继承时子类合并父类公共属性问题 
         final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
         checkMergedBeanDefinition(mbd, beanName, args);

         // Guarantee initialization of beans that the current bean depends on.
         //获取当前Bean所有依赖Bean的名称
         //例如：<bean id = "hw"  class="src.HelloWorld" lazy-init="true"  depends-on="beanA"> ;那么hw必须要在beanA之后实例化
         String[] dependsOn = mbd.getDependsOn();
         //如果当前Bean有依赖Bean;注意这里只是  deponse-on 属性的依赖
          if (dependsOn != null) {
			for (String dep : dependsOn) {
    //1.检查是否有循环依赖；
    //例如：<bean id = "hw"class="HelloWorld" depends-on="beanA">这个hw依赖了 beanA
    //然后beanA又依赖了hw;<bean id="beanA" class="BeanA" depends-on="hw">这样就会抛出异常
				if (isDependent(beanName, dep)) 
                {
					throw new BeanCreationException(mbd.getResourceDescription(), beanName,"Circular depends-on relationship between '" + beanName + "' and '" + dep + "'");
				}
                //例子:<bean id = "hw"  class="src.HelloWorld" depends-on="beanA">
                //标记依赖bean；这里面做了两件事情；
                //第一件是 this.dependentBeanMap.put(canonicalName, dependentBeans);这个dependentBeanMap表示的是 某个bean被哪些bean给依赖了;在上面的配置中；dependentBeanMap存了一个 key为beanA；value为 （Set<String>中有个hw；存了所有需要依赖beanA的beanName）
                //3.第二件事是this.dependenciesForBeanMap.put(dependentBeanName, dependenciesForBean);
                //这个跟2.想反，这个存的是hw需要依赖的所有beanName
				registerDependentBean(dep, beanName);
                //递归调用getBean方法，获取当前Bean的依赖Bean
				getBean(dep);
			}
		}
         // Create bean instance.
         //创建单例模式Bean的实例对象
         if (mbd.isSingleton()) {
            //这里使用了一个匿名内部类，创建Bean实例对象，并且注册给所依赖的对象
            sharedInstance = getSingleton(beanName, new ObjectFactory<Object>() {
               public Object getObject() throws BeansException {
                  try {
                     //创建一个指定Bean实例对象，如果有父级继承，则合并子类和父类的定义(**重点**)
                     return createBean(beanName, mbd, args);
                  }
                  catch (BeansException ex) {
                     //显式地从容器单例模式Bean缓存中清除实例对象
                     destroySingleton(beanName);
                     throw ex;
                  }
               }
            });
            //获取给定Bean的实例对象
            bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
         }

         //IOC容器创建原型模式Bean实例对象
         else if (mbd.isPrototype()) {
            // It's a prototype -> create a new instance.
            //原型模式(Prototype)是每次都会创建一个新的对象
            Object prototypeInstance = null;
            try {
               //回调beforePrototypeCreation方法，默认的功能是注册当前创建的原型对象
               beforePrototypeCreation(beanName);
               //创建指定Bean对象实例
               prototypeInstance = createBean(beanName, mbd, args);
            }
            finally {
               //回调afterPrototypeCreation方法，默认的功能告诉IOC容器指定Bean的原型对象不再创建了
               afterPrototypeCreation(beanName);
            }
            //获取给定Bean的实例对象
            bean = getObjectForBeanInstance(prototypeInstance, name, beanName, mbd);
         }

         //要创建的Bean既不是单例模式，也不是原型模式，则根据Bean定义资源中  
           //配置的生命周期范围，选择实例化Bean的合适方法，这种在Web应用程序中  
           //比较常用，如：request、session、application等生命周期 
         else {
            String scopeName = mbd.getScope();
            final Scope scope = this.scopes.get(scopeName);
            //Bean定义资源中没有配置生命周期范围，则Bean定义不合法
            if (scope == null) {
               throw new IllegalStateException("No Scope registered for scope '" + scopeName + "'");
            }
            try {
               //这里又使用了一个匿名内部类，获取一个指定生命周期范围的实例
               Object scopedInstance = scope.get(beanName, new ObjectFactory<Object>() {
                  public Object getObject() throws BeansException {
                     beforePrototypeCreation(beanName);
                     try {
                        return createBean(beanName, mbd, args);
                     }
                     finally {
                        afterPrototypeCreation(beanName);
                     }
                  }
               });
               //获取给定Bean的实例对象
               bean = getObjectForBeanInstance(scopedInstance, name, beanName, mbd);
            }
            catch (IllegalStateException ex) {
               throw new BeanCreationException(beanName,
                     "Scope '" + scopeName + "' is not active for the current thread; " +
                     "consider defining a scoped proxy for this bean if you intend to refer to it from a singleton",
                     ex);
            }
         }
      }
      catch (BeansException ex) {
         cleanupAfterBeanCreationFailure(beanName);
         throw ex;
      }
   }

   // Check if required type matches the type of the actual bean instance.
   //对创建的Bean实例对象进行类型检查
   if (requiredType != null && bean != null && !requiredType.isAssignableFrom(bean.getClass())) {
      try {
         return getTypeConverter().convertIfNecessary(bean, requiredType);
      }
      catch (TypeMismatchException ex) {
         if (logger.isDebugEnabled()) {
            logger.debug("Failed to convert bean '" + name + "' to required type [" +
                  ClassUtils.getQualifiedName(requiredType) + "]", ex);
         }
         throw new BeanNotOfRequiredTypeException(name, requiredType, bean.getClass());
      }
   }
   return (T) bean;
}
```

**getBean()的实现是依赖注入的起点，之后会调用createBean()，下面我们分析createBean的代码**

### 3.3createBean()方法

> **createBean()的实现是在AbstractAutowireCapableBeanFactory当中**

```java
//AbstractAutowireCapableBeanFactory.java

//创建Bean实例对象
@Override
protected Object createBean(final String beanName, final RootBeanDefinition mbd, final Object[] args)
      throws BeanCreationException {

   if (logger.isDebugEnabled()) {
      logger.debug("Creating instance of bean '" + beanName + "'");
   }
   //1.判断需要创建的Bean是否可以实例化，即是否可以通过当前的类加载器加载
   resolveBeanClass(mbd, beanName);

   // Prepare method overrides.
   //2.校验和准备Bean中的方法覆盖
   try {
      mbd.prepareMethodOverrides();
   }
   catch (BeanDefinitionValidationException ex) {
      throw new BeanDefinitionStoreException(mbd.getResourceDescription(),
            beanName, "Validation of method overrides failed", ex);
   }

   try {
      // Give BeanPostProcessors a chance to return a proxy instead of the target bean instance.
      //让BeanPostProcessors先拦截返回一个代理对象
      //如果Bean配置了初始化前和初始化后的处理器，则试图返回一个需要创建Bean的代理对象
      Object bean = resolveBeforeInstantiation(beanName, mbd);
      if (bean != null) {
         return bean;
      }
   }
   catch (Throwable ex) {
      throw new BeanCreationException(mbd.getResourceDescription(), beanName,
            "BeanPostProcessor before instantiation of bean failed", ex);
   }

   //创建Bean的入口(**重点**)
   Object beanInstance = doCreateBean(beanName, mbd, args);
   if (logger.isDebugEnabled()) {
      logger.debug("Finished creating instance of bean '" + beanName + "'");
   }
   return beanInstance;
}
```

### 3.4 doCreateBean()方法

> **doCreateBean()的实现是在AbstractAutowireCapableBeanFactory当中**

```java
//AbstractAutowireCapableBeanFactory.java

//真正创建Bean的方法 
protected Object doCreateBean(final String beanName, final RootBeanDefinition mbd, final Object[] args) {
   // Instantiate the bean.
   //封装被创建的Bean对象(不是真正的bean)
   BeanWrapper instanceWrapper = null;
   if (mbd.isSingleton()) {//单例模式的Bean，先从容器中缓存中获取同名Bean
      instanceWrapper = this.factoryBeanInstanceCache.remove(beanName);
   }
   if (instanceWrapper == null) {
      //创建实例对象-------------重点(生成bean所包含的java对象实例）
      instanceWrapper = createBeanInstance(beanName, mbd, args);
   }
   final Object bean = (instanceWrapper != null ? instanceWrapper.getWrappedInstance() : null);
   //获取实例化对象的类型
   Class<?> beanType = (instanceWrapper != null ? instanceWrapper.getWrappedClass() : null);

   // Allow post-processors to modify the merged bean definition.
   //调用PostProcessor后置处理器
   synchronized (mbd.postProcessingLock) {
      if (!mbd.postProcessed) {
         applyMergedBeanDefinitionPostProcessors(mbd, beanType, beanName);
         mbd.postProcessed = true;
      }
   }

   // Eagerly cache singletons to be able to resolve circular references
   // even when triggered by lifecycle interfaces like BeanFactoryAware.
   //向容器中缓存单例模式的Bean对象，以防循环引用
   boolean earlySingletonExposure = (mbd.isSingleton() && this.allowCircularReferences &&
         isSingletonCurrentlyInCreation(beanName));
   if (earlySingletonExposure) {
      if (logger.isDebugEnabled()) {
         logger.debug("Eagerly caching bean '" + beanName +
               "' to allow for resolving potential circular references");
      }
      //这里是一个匿名内部类，为了防止循环引用，尽早持有对象的引用
      addSingletonFactory(beanName, new ObjectFactory<Object>() {
         public Object getObject() throws BeansException {
            return getEarlyBeanReference(beanName, mbd, bean);
         }
      });
   }

   // Initialize the bean instance.
   //Bean对象的初始化，依赖注入在此触发  
    //这个exposedObject在初始化完成之后返回作为依赖注入完成后的Bean
   Object exposedObject = bean;
   try {
      //将Bean实例对象封装，并且Bean定义中配置的属性值赋值给实例对象------------------重点（对Bean属性的依赖注入进行处理）
      populateBean(beanName, mbd, instanceWrapper);
      if (exposedObject != null) {
         //初始化Bean对象 
         //在对Bean实例对象生成和依赖注入完成以后，开始对Bean实例对象  
            //进行初始化 ，为Bean实例对象应用BeanPostProcessor后置处理器
         exposedObject = initializeBean(beanName, exposedObject, mbd);
      }
   }
   catch (Throwable ex) {
      if (ex instanceof BeanCreationException && beanName.equals(((BeanCreationException) ex).getBeanName())) {
         throw (BeanCreationException) ex;
      }
      else {
         throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Initialization of bean failed", ex);
      }
   }

   if (earlySingletonExposure) {
      //获取指定名称的已注册的单例模式Bean对象
      Object earlySingletonReference = getSingleton(beanName, false);
      if (earlySingletonReference != null) {
         //根据名称获取的已注册的Bean和正在实例化的Bean是同一个
         if (exposedObject == bean) {
            //当前实例化的Bean初始化完成
            exposedObject = earlySingletonReference;
         }
         //当前Bean依赖其他Bean，并且当发生循环引用时不允许新创建实例对象
         else if (!this.allowRawInjectionDespiteWrapping && hasDependentBean(beanName)) {
            String[] dependentBeans = getDependentBeans(beanName);
            Set<String> actualDependentBeans = new LinkedHashSet<String>(dependentBeans.length);
            //获取当前Bean所依赖的其他Bean 
            for (String dependentBean : dependentBeans) {
               //对依赖Bean进行类型检查
               if (!removeSingletonIfCreatedForTypeCheckOnly(dependentBean)) {
                  actualDependentBeans.add(dependentBean);
               }
            }
            if (!actualDependentBeans.isEmpty()) {
               throw new BeanCurrentlyInCreationException(beanName,
                     "Bean with name '" + beanName + "' has been injected into other beans [" +
                     StringUtils.collectionToCommaDelimitedString(actualDependentBeans) +
                     "] in its raw version as part of a circular reference, but has eventually been " +
                     "wrapped. This means that said other beans do not use the final version of the " +
                     "bean. This is often the result of over-eager type matching - consider using " +
                     "'getBeanNamesOfType' with the 'allowEagerInit' flag turned off, for example.");
            }
         }
      }
   }

   // Register bean as disposable.
   //注册完成依赖注入的Bean
   try {
      registerDisposableBeanIfNecessary(beanName, bean, mbd);
   }
   catch (BeanDefinitionValidationException ex) {
      throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Invalid destruction signature", ex);
   }

   //为应用返回所需要的实例对象
   return exposedObject;
}
```

  

>  看了上面的代码后，我们应该重点关注 `createBeanInstance()` 和 `populateBean()` 这两个方法。其中，createBeanInstance方法生成了Bean所包含的Java对象,依赖关系是如何注入到Bean对象中，就发生在populateBean()方法中。

### 3.5 createBeanInstance()方法

> **createBeanInstance()的实现是在AbstractAutowireCapableBeanFactory当中**

```java
//AbstractAutowireCapableBeanFactory.java

//创建Bean的实例对象  利用工厂方法或对象的构造方法构造出对象
protected BeanWrapper createBeanInstance(String beanName, RootBeanDefinition mbd, Object[] args) {
   // Make sure bean class is actually resolved at this point.
   //1. 检查确认Bean是可实例化的
   Class<?> beanClass = resolveBeanClass(mbd, beanName);

   //2.使用工厂方法对Bean进行实例化
   if (beanClass != null && !Modifier.isPublic(beanClass.getModifiers()) && !mbd.isNonPublicAccessAllowed()) {
      throw new BeanCreationException(mbd.getResourceDescription(), beanName,
            "Bean class isn't public, and non-public access not allowed: " + beanClass.getName());
   }

   if (mbd.getFactoryMethodName() != null)  {
      //调用工厂方法实例化
      return instantiateUsingFactoryMethod(beanName, mbd, args);
   }

   // Shortcut when re-creating the same bean...
   //使用容器的自动装配方法进行实例化
   boolean resolved = false;
   boolean autowireNecessary = false;
   if (args == null) {
      synchronized (mbd.constructorArgumentLock) {
         if (mbd.resolvedConstructorOrFactoryMethod != null) {
            resolved = true;
            autowireNecessary = mbd.constructorArgumentsResolved;
         }
      }
   }
   if (resolved) {
      if (autowireNecessary) {
         //配置了自动装配属性，使用容器的自动装配实例化  
            //容器的自动装配是根据参数类型匹配Bean的构造方法
         return autowireConstructor(beanName, mbd, null, null);
      }
      else {
         //使用默认的无参构造方法实例化
         return instantiateBean(beanName, mbd);
      }
   }

   // Need to determine the constructor...
   //使用Bean的构造方法进行实例化
   Constructor<?>[] ctors = determineConstructorsFromBeanPostProcessors(beanClass, beanName);
   if (ctors != null ||
         mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_CONSTRUCTOR ||
         mbd.hasConstructorArgumentValues() || !ObjectUtils.isEmpty(args))  {
      //使用容器的自动装配特性，调用匹配的构造方法实例化 
      return autowireConstructor(beanName, mbd, ctors, args);
   }

   // No special handling: simply use no-arg constructor.
   //使用默认的无参构造方法实例化
   return instantiateBean(beanName, mbd);
}
```

### 3.6 instantiateBean()方法

> **instantiateBean()的实现是在AbstractAutowireCapableBeanFactory当中**

```java
//AbstractAutowireCapableBeanFactory.java

//使用默认的无参构造方法实例化Bean对象
protected BeanWrapper instantiateBean(final String beanName, final RootBeanDefinition mbd) {
   try {
      Object beanInstance;
      final BeanFactory parent = this;
      //获取系统的安全管理接口，JDK标准的安全管理AP
      if (System.getSecurityManager() != null) {
         //这里是一个匿名内置类，根据实例化策略创建实例对象
         beanInstance = AccessController.doPrivileged(new PrivilegedAction<Object>() {
            public Object run() {
               return getInstantiationStrategy().instantiate(mbd, beanName, parent);
            }
         }, getAccessControlContext());
      }
      else {
         //将实例化的对象封装起来
         beanInstance = getInstantiationStrategy().instantiate(mbd, beanName, parent);
      }
      BeanWrapper bw = new BeanWrapperImpl(beanInstance);
      initBeanWrapper(bw);
      return bw;
   }
   catch (Throwable ex) {
      throw new BeanCreationException(mbd.getResourceDescription(), beanName, "Instantiation of bean failed", ex);
   }
}
```

 > 我们重点关注 `getInstantiationStrategy()` 这个方法，可以看到instantiateBean方法的功能实现是通过调用getInstantiationStrategy().instantiate方法实现的。 **getInstantiationStrategy** 方法的作用是获得实例化的策略对象，也就是指通过哪种方案进行实例化的过程。继续跟踪下去我们可以发现，Spring当中提供了两种实例化方案： **BeanUtils** 和 **Cglib** 方式。BeanUtils实现机制是通过Java的反射机制，Cglib是一个第三方类库采用的是一种字节码加强方式机制。 *Spring中采用的默认实例化策略是Cglib。* 
 >   分析到这里我们已经知道了实例化Bean对象的流程，现在已经是万事具备，只欠东风，就剩下对这些建立好的Bean对象建立联系了。

### 3.7populateBean()方法

> **我们要分析依赖关系是怎样注入到Bean对象当中，就需要回到前面的 `populateBean()` 方法，这个方法实现是在 *AbstractAutowireCapableBeanFactory*中**

```java
//AbstractAutowireCapableBeanFactory.java

//将Bean属性设置到生成的实例对象上
protected void populateBean(String beanName, RootBeanDefinition mbd, BeanWrapper bw) {
   //获取容器在解析Bean定义资源时为BeanDefiniton中设置的属性值
   PropertyValues pvs = mbd.getPropertyValues();

   //实例对象为null
   if (bw == null) {
      if (!pvs.isEmpty()) {
         throw new BeanCreationException(
               mbd.getResourceDescription(), beanName, "Cannot apply property values to null instance");
      }
      else {
         // Skip property population phase for null instance.
         //实例对象为null，属性值也为空，不需要设置属性值，直接返回 
         return;
      }
   }

   // Give any InstantiationAwareBeanPostProcessors the opportunity to modify the
   // state of the bean before properties are set. This can be used, for example,
   // to support styles of field injection.
   //在设置属性之前调用Bean的PostProcessor后置处理器
   boolean continueWithPropertyPopulation = true;

   if (!mbd.isSynthetic() && hasInstantiationAwareBeanPostProcessors()) {
      for (BeanPostProcessor bp : getBeanPostProcessors()) {
         if (bp instanceof InstantiationAwareBeanPostProcessor) {
            InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
            if (!ibp.postProcessAfterInstantiation(bw.getWrappedInstance(), beanName)) {
               continueWithPropertyPopulation = false;
               break;
            }
         }
      }
   }

   if (!continueWithPropertyPopulation) {
      return;
   }

   //依赖注入开始，首先处理autowire自动装配的注入
   if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME ||
         mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
      MutablePropertyValues newPvs = new MutablePropertyValues(pvs);

      // Add property values based on autowire by name if applicable.
      //对autowire自动装配的处理，根据Bean名称自动装配注入
      if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_NAME) {
         autowireByName(beanName, mbd, bw, newPvs);
      }

      // Add property values based on autowire by type if applicable.
      //根据Bean类型自动装配注入
      if (mbd.getResolvedAutowireMode() == RootBeanDefinition.AUTOWIRE_BY_TYPE) {
         autowireByType(beanName, mbd, bw, newPvs);
      }

      pvs = newPvs;
   }

   //检查容器是否持有用于处理单例模式Bean关闭时的后置处理器
   boolean hasInstAwareBpps = hasInstantiationAwareBeanPostProcessors();
   //Bean实例对象没有依赖，即没有继承基类
   boolean needsDepCheck = (mbd.getDependencyCheck() != RootBeanDefinition.DEPENDENCY_CHECK_NONE);

   if (hasInstAwareBpps || needsDepCheck) {
      //从实例对象中提取属性描述符
      PropertyDescriptor[] filteredPds = filterPropertyDescriptorsForDependencyCheck(bw, mbd.allowCaching);
      if (hasInstAwareBpps) {
         for (BeanPostProcessor bp : getBeanPostProcessors()) {
            if (bp instanceof InstantiationAwareBeanPostProcessor) {
               InstantiationAwareBeanPostProcessor ibp = (InstantiationAwareBeanPostProcessor) bp;
               //使用BeanPostProcessor处理器处理属性值
               pvs = ibp.postProcessPropertyValues(pvs, filteredPds, bw.getWrappedInstance(), beanName);
               if (pvs == null) {
                  return;
               }
            }
         }
      }
      if (needsDepCheck) {
         //为要设置的属性进行依赖检查
         checkDependencies(beanName, mbd, filteredPds, pvs);
      }
   }
   //对属性进行注入
   applyPropertyValues(beanName, mbd, bw, pvs);
}
```

### 3.8 applyPropertyValue()方法

> **applyPropertyValue()的实现是在AbstractAutowireCapableBeanFactory当中**

```java
//AbstractAutowireCapableBeanFactory.java

//解析并注入依赖属性的过程
protected void applyPropertyValues(String beanName, BeanDefinition mbd, BeanWrapper bw, PropertyValues pvs) {
   if (pvs == null || pvs.isEmpty()) {
      return;
   }

   //封装属性值
   MutablePropertyValues mpvs = null;
   List<PropertyValue> original;

   if (System.getSecurityManager()!= null) {
      if (bw instanceof BeanWrapperImpl) {
         //设置安全上下文，JDK安全机制
         ((BeanWrapperImpl) bw).setSecurityContext(getAccessControlContext());
      }
   }

   if (pvs instanceof MutablePropertyValues) {
      mpvs = (MutablePropertyValues) pvs;
      //属性值已经转换
      if (mpvs.isConverted()) {
         // Shortcut: use the pre-converted values as-is.
         try {
            //为实例化对象设置属性值
            bw.setPropertyValues(mpvs);
            return;
         }
         catch (BeansException ex) {
            throw new BeanCreationException(
                  mbd.getResourceDescription(), beanName, "Error setting property values", ex);
         }
      }
      //获取属性值对象的原始类型值
      original = mpvs.getPropertyValueList();
   }
   else {
      original = Arrays.asList(pvs.getPropertyValues());
   }

   //获取用户自定义的类型转换
   TypeConverter converter = getCustomTypeConverter();
   if (converter == null) {
      converter = bw;
   }
   //创建一个Bean定义属性值解析器，将Bean定义中的属性值解析为Bean实例对象的实际值
   BeanDefinitionValueResolver valueResolver = new BeanDefinitionValueResolver(this, beanName, mbd, converter);

   // Create a deep copy, resolving any references for values.
   //为属性的解析值创建一个拷贝，将拷贝的数据注入到实例对象中
   List<PropertyValue> deepCopy = new ArrayList<PropertyValue>(original.size());
   boolean resolveNecessary = false;
   for (PropertyValue pv : original) {
      //属性值不需要转换
      if (pv.isConverted()) {
         deepCopy.add(pv);
      }
      //属性值需要转换
      else {
         String propertyName = pv.getName();
         //原始的属性值，即转换之前的属性值
         Object originalValue = pv.getValue();
         //转换属性值，例如将引用转换为IoC容器中实例化对象引用
         Object resolvedValue = valueResolver.resolveValueIfNecessary(pv, originalValue);
         //转换之后的属性值
         Object convertedValue = resolvedValue;
         //属性值是否可以转换
         boolean convertible = bw.isWritableProperty(propertyName) &&
               !PropertyAccessorUtils.isNestedOrIndexedProperty(propertyName);
         if (convertible) {
            //使用用户自定义的类型转换器转换属性值
            convertedValue = convertForProperty(resolvedValue, propertyName, bw, converter);
         }
         // Possibly store converted value in merged bean definition,
         // in order to avoid re-conversion for every created bean instance.
         //存储转换后的属性值，避免每次属性注入时的转换工作
         if (resolvedValue == originalValue) {
            if (convertible) {
               //设置属性转换之后的值
               pv.setConvertedValue(convertedValue);
            }
            deepCopy.add(pv);
         }
         //属性是可转换的，且属性原始值是字符串类型，且属性的原始类型值不是  
            //动态生成的字符串，且属性的原始值不是集合或者数组类型
         else if (convertible && originalValue instanceof TypedStringValue &&
               !((TypedStringValue) originalValue).isDynamic() &&
               !(convertedValue instanceof Collection || ObjectUtils.isArray(convertedValue))) {
            pv.setConvertedValue(convertedValue);
            deepCopy.add(pv);
         }
         else {
            resolveNecessary = true;
            //重新封装属性的值
            deepCopy.add(new PropertyValue(pv, convertedValue));
         }
      }
   }
   if (mpvs != null && !resolveNecessary) {
      //标记属性值已经转换过
      mpvs.setConverted();
   }

 // Set our (possibly massaged) deep copy.
 //进行属性依赖注入(也就是依赖注入发生的地方，BeanWrapperImpl中实现了这个setPropertyValues方法)
   try {
      bw.setPropertyValues(new MutablePropertyValues(deepCopy));
   }
   catch (BeansException ex) {
      throw new BeanCreationException(
            mbd.getResourceDescription(), beanName, "Error setting property values", ex);
   }
}
```

> 上面的代码中通过 **BeanDefinitionResolver** 来对BeanDefinition进行解析，我们可以看下该上面解析过程中调用的主要处理方法`resolveValueIfNecessary()`。



### 3.9 resolveVauleIfNecessary()

> **resolveVauleIfNecessary()的实现是在BeanDefinitionValueResolver当中**

```java
//BeanDefinitionValueResolver.java


//解析属性值，对注入类型进行转换
public Object resolveValueIfNecessary(Object argName, Object value) {
   // We must check each value to see whether it requires a runtime reference
   // to another bean to be resolved.
   //对引用类型的属性进行解析
   if (value instanceof RuntimeBeanReference) {
      RuntimeBeanReference ref = (RuntimeBeanReference) value;
      //调用引用类型属性的解析方法
      return resolveReference(argName, ref);
   }
   //对属性值是引用容器中另一个Bean名称的解析
   else if (value instanceof RuntimeBeanNameReference) {
      String refName = ((RuntimeBeanNameReference) value).getBeanName();
      refName = String.valueOf(evaluate(refName));
      //从容器中获取指定名称的Bean
      if (!this.beanFactory.containsBean(refName)) {
         throw new BeanDefinitionStoreException(
               "Invalid bean name '" + refName + "' in bean reference for " + argName);
      }
      return refName;
   }
   //对Bean类型属性的解析，主要是Bean中的内部类
   else if (value instanceof BeanDefinitionHolder) {
      // Resolve BeanDefinitionHolder: contains BeanDefinition with name and aliases.
      BeanDefinitionHolder bdHolder = (BeanDefinitionHolder) value;
      return resolveInnerBean(argName, bdHolder.getBeanName(), bdHolder.getBeanDefinition());
   }
   else if (value instanceof BeanDefinition) {
      // Resolve plain BeanDefinition, without contained name: use dummy name.
      BeanDefinition bd = (BeanDefinition) value;
      return resolveInnerBean(argName, "(inner bean)", bd);
   }
   //对集合数组类型的属性解析
   else if (value instanceof ManagedArray) {
      // May need to resolve contained runtime references.
      ManagedArray array = (ManagedArray) value;
      //获取数组的类型
      Class<?> elementType = array.resolvedElementType;
      if (elementType == null) {
          //获取数组元素的类型
         String elementTypeName = array.getElementTypeName();
         if (StringUtils.hasText(elementTypeName)) {
            try {
               //使用反射机制创建指定类型的对象
               elementType = ClassUtils.forName(elementTypeName, this.beanFactory.getBeanClassLoader());
               array.resolvedElementType = elementType;
            }
            catch (Throwable ex) {
               // Improve the message by showing the context.
               throw new BeanCreationException(
                     this.beanDefinition.getResourceDescription(), this.beanName,
                     "Error resolving array type for " + argName, ex);
            }
         }
         //没有获取到数组的类型，也没有获取到数组元素的类型 
            //则直接设置数组的类型为Object
         else {
            elementType = Object.class;
         }
      }
      //创建指定类型的数组
      return resolveManagedArray(argName, (List<?>) value, elementType);
   }
   //解析list类型的属性值
   else if (value instanceof ManagedList) {
      // May need to resolve contained runtime references.
      return resolveManagedList(argName, (List<?>) value);
   }
   //解析set类型的属性值
   else if (value instanceof ManagedSet) {
      // May need to resolve contained runtime references.
      return resolveManagedSet(argName, (Set<?>) value);
   }
   //解析map类型的属性值
   else if (value instanceof ManagedMap) {
      // May need to resolve contained runtime references.
      return resolveManagedMap(argName, (Map<?, ?>) value);
   }
   //解析props类型的属性值，props其实就是key和value均为字符串的map
   else if (value instanceof ManagedProperties) {
      Properties original = (Properties) value;
      //创建一个拷贝，用于作为解析后的返回值
      Properties copy = new Properties();
      for (Map.Entry propEntry : original.entrySet()) {
         Object propKey = propEntry.getKey();
         Object propValue = propEntry.getValue();
         if (propKey instanceof TypedStringValue) {
            propKey = evaluate((TypedStringValue) propKey);
         }
         if (propValue instanceof TypedStringValue) {
            propValue = evaluate((TypedStringValue) propValue);
         }
         copy.put(propKey, propValue);
      }
      return copy;
   }
   //解析字符串类型的属性值
   else if (value instanceof TypedStringValue) {
      // Convert value to target type here.
      TypedStringValue typedStringValue = (TypedStringValue) value;
      Object valueObject = evaluate(typedStringValue);
      try {
         //获取属性的目标类型
         Class<?> resolvedTargetType = resolveTargetType(typedStringValue);
         if (resolvedTargetType != null) {
            //对目标类型的属性进行解析，递归调用
            return this.typeConverter.convertIfNecessary(valueObject, resolvedTargetType);
         }
         //没有获取到属性的目标对象，则按Object类型返回
         else {
            return valueObject;
         }
      }
      catch (Throwable ex) {
         // Improve the message by showing the context.
         throw new BeanCreationException(
               this.beanDefinition.getResourceDescription(), this.beanName,
               "Error converting typed String value for " + argName, ex);
      }
   }
   else {
      return evaluate(value);
   }
}
```

>   在配置Bean的属性的时候，属性可能有多种类型，我们再进行注入的时候，不同的属性类型我们不可能一概而论的进行处理，集合类型的属性和非集合类型具备很大的差别，对不同的类型应该有不同的解析处理过程，故该方法流程中首先判断value的类型然后在分别调用 `resolveManagedList()`、 `resolveManagedList()` 、 `resolveManagedList()`等方法进行具体的解析。为了掌握大体的主线，这里我们就不详细展示这几个方法源码，感兴趣的可以自己下来分析。
>
>   在完成这个解析过程后，已经为依赖注入准备好了条件，上文中applyPropertyValues()方法的 `setPropertyValue()` 方法是真正设置属性依赖的地方，该方法的实现是在BeanWrapper的实现类BeanWrapperImpl中

### 3.10 setPropertyValue()方法

> **setPropertyValue()方法在BeanWrapperImpl中**

```java
//BeanWrapperImpl.java


//实现属性依赖注入功能
@SuppressWarnings("unchecked")
private void setPropertyValue(PropertyTokenHolder tokens, PropertyValue pv) throws BeansException {
   //PropertyTokenHolder主要保存属性的名称、路径，以及集合的size等信息
   String propertyName = tokens.canonicalName;
   String actualName = tokens.actualName;

   //keys是用来保存集合类型属性的size
   if (tokens.keys != null) {
      // Apply indexes and map keys: fetch value for all keys but the last one.
      //将属性信息拷贝
      PropertyTokenHolder getterTokens = new PropertyTokenHolder();
      getterTokens.canonicalName = tokens.canonicalName;
      getterTokens.actualName = tokens.actualName;
      getterTokens.keys = new String[tokens.keys.length - 1];
      System.arraycopy(tokens.keys, 0, getterTokens.keys, 0, tokens.keys.length - 1);
      Object propValue;
      try {
         //获取属性值，该方法内部使用JDK的内省(Introspector)机制
            //调用属性的getter(readerMethod)方法，获取属性的值 
         propValue = getPropertyValue(getterTokens);
      }
      catch (NotReadablePropertyException ex) {
         throw new NotWritablePropertyException(getRootClass(), this.nestedPath + propertyName,
               "Cannot access indexed value in property referenced " +
               "in indexed property path '" + propertyName + "'", ex);
      }
      // Set value for last key.
      //获取集合类型属性的长度
      String key = tokens.keys[tokens.keys.length - 1];
      if (propValue == null) {
         // null map value case
         if (this.autoGrowNestedPaths) {
            // TODO: cleanup, this is pretty hacky
            int lastKeyIndex = tokens.canonicalName.lastIndexOf('[');
            getterTokens.canonicalName = tokens.canonicalName.substring(0, lastKeyIndex);
            propValue = setDefaultValue(getterTokens);
         }
         else {
            throw new NullValueInNestedPathException(getRootClass(), this.nestedPath + propertyName,
                  "Cannot access indexed value in property referenced " +
                  "in indexed property path '" + propertyName + "': returned null");
         }
      }
      //注入array类型的属性值
      if (propValue.getClass().isArray()) {
         //获取属性的描述符
         PropertyDescriptor pd = getCachedIntrospectionResults().getPropertyDescriptor(actualName);
         //获取数组的类型
         Class requiredType = propValue.getClass().getComponentType();
         //获取数组的长度
         int arrayIndex = Integer.parseInt(key);
         Object oldValue = null;
         try {
            //获取数组以前初始化的值
            if (isExtractOldValueForEditor() && arrayIndex < Array.getLength(propValue)) {
               oldValue = Array.get(propValue, arrayIndex);
            }
            //将属性的值赋值给数组中的元素
            Object convertedValue = convertIfNecessary(propertyName, oldValue, pv.getValue(),
                  requiredType, TypeDescriptor.nested(property(pd), tokens.keys.length));
            Array.set(propValue, arrayIndex, convertedValue);
         }
         catch (IndexOutOfBoundsException ex) {
            throw new InvalidPropertyException(getRootClass(), this.nestedPath + propertyName,
                  "Invalid array index in property path '" + propertyName + "'", ex);
         }
      }
      //注入list类型的属性值
      else if (propValue instanceof List) {
         PropertyDescriptor pd = getCachedIntrospectionResults().getPropertyDescriptor(actualName);
         //获取list集合的类型
         Class requiredType = GenericCollectionTypeResolver.getCollectionReturnType(
               pd.getReadMethod(), tokens.keys.length);
         List list = (List) propValue;
         
         int index = Integer.parseInt(key);
         Object oldValue = null;
         if (isExtractOldValueForEditor() && index < list.size()) {
            oldValue = list.get(index);
         }
         //获取list解析后的属性值
         Object convertedValue = convertIfNecessary(propertyName, oldValue, pv.getValue(),
               requiredType, TypeDescriptor.nested(property(pd), tokens.keys.length));
         //获取list集合的size
         int size = list.size();
         //如果list的长度大于属性值的长度，则多余的元素赋值为null 
         if (index >= size && index < this.autoGrowCollectionLimit) {
            for (int i = size; i < index; i++) {
               try {
                  list.add(null);
               }
               catch (NullPointerException ex) {
                  throw new InvalidPropertyException(getRootClass(), this.nestedPath + propertyName,
                        "Cannot set element with index " + index + " in List of size " +
                        size + ", accessed using property path '" + propertyName +
                        "': List does not support filling up gaps with null elements");
               }
            }
            list.add(convertedValue);
         }
         else {
            try {
               //为list属性赋值
               list.set(index, convertedValue);
            }
            catch (IndexOutOfBoundsException ex) {
               throw new InvalidPropertyException(getRootClass(), this.nestedPath + propertyName,
                     "Invalid list index in property path '" + propertyName + "'", ex);
            }
         }
      }
      //注入map类型的属性值
      else if (propValue instanceof Map) {
         PropertyDescriptor pd = getCachedIntrospectionResults().getPropertyDescriptor(actualName);
         //获取map集合key的类型
         Class mapKeyType = GenericCollectionTypeResolver.getMapKeyReturnType(
               pd.getReadMethod(), tokens.keys.length);
         //获取map集合value的类型
         Class mapValueType = GenericCollectionTypeResolver.getMapValueReturnType(
               pd.getReadMethod(), tokens.keys.length);
         Map map = (Map) propValue;
         // IMPORTANT: Do not pass full property name in here - property editors
         // must not kick in for map keys but rather only for map values.
         TypeDescriptor typeDescriptor = (mapKeyType != null ?
               TypeDescriptor.valueOf(mapKeyType) : TypeDescriptor.valueOf(Object.class));
         //解析map类型属性key值
         Object convertedMapKey = convertIfNecessary(null, null, key, mapKeyType, typeDescriptor);
         Object oldValue = null;
         if (isExtractOldValueForEditor()) {
            oldValue = map.get(convertedMapKey);
         }
         // Pass full property name and old value in here, since we want full
         // conversion ability for map values.
         //解析map类型属性value值
         Object convertedMapValue = convertIfNecessary(propertyName, oldValue, pv.getValue(),
               mapValueType, TypeDescriptor.nested(property(pd), tokens.keys.length));
         //将解析后的key和value值赋值给map集合属性
         map.put(convertedMapKey, convertedMapValue);
      }
      //对非集合类型的属性注入
      else {
         throw new InvalidPropertyException(getRootClass(), this.nestedPath + propertyName,
               "Property referenced in indexed property path '" + propertyName +
               "' is neither an array nor a List nor a Map; returned value was [" + pv.getValue() + "]");
      }
   }

   else {
      PropertyDescriptor pd = pv.resolvedDescriptor;
      if (pd == null || !pd.getWriteMethod().getDeclaringClass().isInstance(this.object)) {
         pd = getCachedIntrospectionResults().getPropertyDescriptor(actualName);
         //无法获取到属性名或者属性没有提供setter(写方法)方法
         if (pd == null || pd.getWriteMethod() == null) {
            //如果属性值是可选的，即不是必须的，则忽略该属性值
            if (pv.isOptional()) {
               logger.debug("Ignoring optional value for property '" + actualName +
                     "' - property not found on bean class [" + getRootClass().getName() + "]");
               return;
            }
            //如果属性值是必须的，则抛出无法给属性赋值，因为没提供setter方法异常
            else {
               PropertyMatches matches = PropertyMatches.forProperty(propertyName, getRootClass());
               throw new NotWritablePropertyException(
                     getRootClass(), this.nestedPath + propertyName,
                     matches.buildErrorMessage(), matches.getPossibleMatches());
            }
         }
         pv.getOriginalPropertyValue().resolvedDescriptor = pd;
      }

      Object oldValue = null;
      try {
         Object originalValue = pv.getValue();
         Object valueToApply = originalValue;
         if (!Boolean.FALSE.equals(pv.conversionNecessary)) {
            if (pv.isConverted()) {
               valueToApply = pv.getConvertedValue();
            }
            else {
               if (isExtractOldValueForEditor() && pd.getReadMethod() != null) {
                  //获取属性的getter方法(读方法)，JDK内省机制
                  final Method readMethod = pd.getReadMethod();
                  //如果属性的getter方法不是public访问控制权限的，即访问控制权限比较严格，  
                        //则使用JDK的反射机制强行访问非public的方法(暴力读取属性值) 
                  if (!Modifier.isPublic(readMethod.getDeclaringClass().getModifiers()) &&
                        !readMethod.isAccessible()) {
                     if (System.getSecurityManager()!= null) {
                        //匿名内部类，根据权限修改属性的读取控制限制 
                        AccessController.doPrivileged(new PrivilegedAction<Object>() {
                           public Object run() {
                              readMethod.setAccessible(true);
                              return null;
                           }
                        });
                     }
                     else {
                        readMethod.setAccessible(true);
                     }
                  }
                  try {
                      //属性没有提供getter方法时，调用潜在的读取属性值的方法，获取属性值 
                     if (System.getSecurityManager() != null) {
                        oldValue = AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
                           public Object run() throws Exception {
                              return readMethod.invoke(object);
                           }
                        }, acc);
                     }
                     else {
                        oldValue = readMethod.invoke(object);
                     }
                  }
                  catch (Exception ex) {
                     if (ex instanceof PrivilegedActionException) {
                        ex = ((PrivilegedActionException) ex).getException();
                     }
                     if (logger.isDebugEnabled()) {
                        logger.debug("Could not read previous value of property '" +
                              this.nestedPath + propertyName + "'", ex);
                     }
                  }
               }
               //设置属性的注入值
               valueToApply = convertForProperty(propertyName, oldValue, originalValue, pd);
            }
            pv.getOriginalPropertyValue().conversionNecessary = (valueToApply != originalValue);
         }
         //根据JDK的内省机制，获取属性的setter(写方法)方法
         final Method writeMethod = (pd instanceof GenericTypeAwarePropertyDescriptor ?
               ((GenericTypeAwarePropertyDescriptor) pd).getWriteMethodForActualAccess() :
               pd.getWriteMethod());
         //如果属性的setter方法是非public，即访问控制权限比较严格，则使用JDK的反射机制，  
            //强行设置setter方法可访问(暴力为属性赋值)  
         if (!Modifier.isPublic(writeMethod.getDeclaringClass().getModifiers()) && !writeMethod.isAccessible()) {
            if (System.getSecurityManager()!= null) {
               AccessController.doPrivileged(new PrivilegedAction<Object>() {
                  public Object run() {
                     writeMethod.setAccessible(true);
                     return null;
                  }
               });
            }
            else {
               writeMethod.setAccessible(true);
            }
         }
         final Object value = valueToApply;
          //如果使用了JDK的安全机制，则需要权限验证 
         if (System.getSecurityManager() != null) {
            try {
               //将属性值设置到属性上去 
               AccessController.doPrivileged(new PrivilegedExceptionAction<Object>() {
                  public Object run() throws Exception {
                     writeMethod.invoke(object, value);
                     return null;
                  }
               }, acc);
            }
            catch (PrivilegedActionException ex) {
               throw ex.getException();
            }
         }
         else {
            writeMethod.invoke(this.object, value);
         }
      }
      catch (TypeMismatchException ex) {
         throw ex;
      }
      catch (InvocationTargetException ex) {
         PropertyChangeEvent propertyChangeEvent =
               new PropertyChangeEvent(this.rootObject, this.nestedPath + propertyName, oldValue, pv.getValue());
         if (ex.getTargetException() instanceof ClassCastException) {
            throw new TypeMismatchException(propertyChangeEvent, pd.getPropertyType(), ex.getTargetException());
         }
         else {
            throw new MethodInvocationException(propertyChangeEvent, ex.getTargetException());
         }
      }
      catch (Exception ex) {
         PropertyChangeEvent pce =
               new PropertyChangeEvent(this.rootObject, this.nestedPath + propertyName, oldValue, pv.getValue());
         throw new MethodInvocationException(pce, ex);
      }
   }
}
```



## 4.总结

1. getBean(beanName);

2. doGetBean(name, null, null, false);

3. 先获取缓存中保存的单实例Bean, 如果能获取到说明这个Bean之前被创建过（所有创建过的单实例Bean都会被缓存起来）（从 private final Map<String, Object> singletonObjects = new ConcurrentHashMap<String, Object> 中获取）；

4. 缓存中取不到，开始Bean的创建对象流程；

5. 标记当前bean已经被创建；

6. 获取Bean的定义信息；

7. 获取当前Bean依赖的其他Bean; 如果有依赖， 按照getBean()把依赖的Bean先创建出来；

8. 启动单实例Bean的创建流程；

   1. createBean(beanName, mbd, args);

   2. Object beanInstance = doCreateBean(beanName, mbdToUse, args); 创建bean;

      1. **创建Bean实例** :createBeanInstance(beanName, mbd, args); 利用工厂方法或者对象的构造器创建出Bean实例
      2. **Bean属性赋值** : populateBean(beanName, bmd, instanceWrapper);
         1. 在applyPropertyValues(beanName, mbd, bw, pvs)中利用setter方法为应用Bean属性赋值； 
      3. **Bean初始化** ：initializeBean(beanName, mbd, bw, pvs);
      4. **保存bean**
