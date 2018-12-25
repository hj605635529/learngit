# cglib源码分析

[TOC]

## 1.cglib动态代理示例

```java
//被代理类
public class Target {
    public void sayhello() {
        System.out.println("hello ......");
    }

    public void add() {
        System.out.println("Target add()");
    }
}

//拦截对象
public class Interceptor implements MethodInterceptor {

	/**
	 * 
	 * @param o   代理对象
	 * @param method    目标对象的方法
	 * @param objects   方法参数
	 * @param methodProxy   代理方法对象
	 * @return
	 * @throws Throwable
	 */
	@Override
	public Object intercept(Object o, Method method, Object[] objects, MethodProxy methodProxy) throws Throwable {
		System.out.println("before intercept.......");
		methodProxy.invokeSuper(o, objects);
		System.out.println("after intercept........");
		return null;
	}
}


//测试类
public class CglibTest {
	public static void main(String[] args) {

		//设置DebuggingClassWriter.DEBUG_LOCATION_PROPERTY的属性值来获取cglib生成的代理类
		System.setProperty(DebuggingClassWriter.DEBUG_LOCATION_PROPERTY, "/home/huangjia/java/testdemo/target/classes/com/cglib");

		//创建一个增强类对象
		Enhancer enhancer = new Enhancer();
		//设置目标类
		enhancer.setSuperclass(Target.class);
		//设置拦截对象
		enhancer.setCallback(new Interceptor()); //通过setCallback会将拦截器对象放入CGLIB$THREAD_CALLBACKS中
        
		//创建代理对象
		Target proxy = (Target) enhancer.create();
		//使用代理对象调用方法
		proxy.sayhello();
		proxy.add();
	}
}

```

![选区_741.png](https://i.loli.net/2018/12/26/5c225c5c4ef74.png)

图中方框里的那个.class就是我们代理类，我们使用idea直接打开。

## 2.源码分析

```java
Target proxy = (Target) enhancer.create();
```

调用createHelper()--------》然后会调用super.create(key)--------》return obj instanceof Class ? this.firstInstance((Class)obj) : this.nextInstance(obj);

firstInstance方法和nextInstance为抽象类AbstractClassGenerator的一个方法，签名如下：

abstract protected Object nextInstance(Object instance) throws Exception;

在子类Enhancer中实现，实现源码如下：

```java
public Object newInstance(Class[] argumentTypes, Object[] arguments, Callback[] callbacks) {
    this.setThreadCallbacks(callbacks); //拦截对象set

    Object var4;
    try {
        if (this.primaryConstructorArgTypes == argumentTypes || Arrays.equals(this.primaryConstructorArgTypes, argumentTypes)) {
            var4 = ReflectUtils.newInstance(this.primaryConstructor, arguments);
            return var4;
        }

        var4 = ReflectUtils.newInstance(this.generatedClass, argumentTypes, arguments);
    } finally {
        this.setThreadCallbacks((Callback[])null);
    }

    return var4;
}
```

```java
public static Object newInstance(Constructor cstruct, Object[] args) {
    boolean flag = cstruct.isAccessible();

    Object var4;
    try {
        if (!flag) {
            cstruct.setAccessible(true);
        }

        Object result = cstruct.newInstance(args); //创建代理对象了
        var4 = result;
    } catch (InstantiationException var10) {
        throw new CodeGenerationException(var10);
    } catch (IllegalAccessException var11) {
        throw new CodeGenerationException(var11);
    } catch (InvocationTargetException var12) {
        throw new CodeGenerationException(var12.getTargetException());
    } finally {
        if (!flag) {
            cstruct.setAccessible(flag);
        }

    }

    return var4;
}
```

代理类（Target$$EnhancerByCGLIB$$f64c12a7）继承了目标类（Target），至于代理类实现的factory接口与本文无关，残忍无视。代理类为每个目标类的方法生成两个方法，例如针对目标类中的每个非private方法，代理类会生成两个方法，以sayhello方法为例：一个是@Override的sayhello()方法，一个是CGLIB$sayhello$0()。我们在示例代码中调用目标类的方法proxy.sayhello()时，实际上调用的是代理类中sayhello()方法。接下来我们着重分析代理类中的sayhello()方法，看看是怎么实现的代理功能。

```java
//代理类继承了目标类  --下面是我复制粘贴了一部分代码
public class Target$$EnhancerByCGLIB$$f64c12a7 extends Target implements Factory {
    private boolean CGLIB$BOUND;
    private static final ThreadLocal CGLIB$THREAD_CALLBACKS;
    private static final Callback[] CGLIB$STATIC_CALLBACKS;
    
    private MethodInterceptor CGLIB$CALLBACK_0; //拦截器对象 通过从CGLIB$THREAD_CALLBACKS或从CGLIB$STATIC_CALLBACKS中获取
    
    private static final Method CGLIB$sayhello$0$Method; //被代理方法
    private static final MethodProxy CGLIB$sayhello$0$Proxy; //代理方法
    
    private static final Object[] CGLIB$emptyArgs;
    
    //	methodProxy.invokeSuper(o, objects);会调用这个方法
    final void CGLIB$sayhello$0() {
        super.sayhello();
    }

    //当用代理对象去调用sayhello()方法时，会调用这个方法
    public final void sayhello() {
        //先判断是否已经存在实现了MethodInterceptor接口的拦截对象
        MethodInterceptor var10000 = this.CGLIB$CALLBACK_0;
        if (this.CGLIB$CALLBACK_0 == null) {
            //调用CGLIB_BIND_CALLBACKS方法来获取拦截对象，这个方法的具体实现，抄了下网上的
            CGLIB$BIND_CALLBACKS(this);
            var10000 = this.CGLIB$CALLBACK_0;
        }
        if (var10000 != null) {
            //调用我们拦截对象的intercept方法了
            var10000.intercept(this, CGLIB$sayhello$0$Method, CGLIB$emptyArgs, CGLIB$sayhello$0$Proxy);
        } else {
            super.sayhello();
        }
    }
    
    ///////////////////////////////////////////////////////////////////////
   //   CGLIB$BIND_CALLBACKS(this);  实现
    private static final void CGLIB$BIND_CALLBACKS(Object o){
        Target$$EnhancerByCGLIB$$788444a0 temp_1 = (Target$$EnhancerByCGLIB$$788444a0)o;
        Object temp_2;
        Callback[] temp_3
        if(temp_1.CGLIB$BOUND == true){
            return;
        }
        temp_1.CGLIB$BOUND = true;
        temp_2 = CGLIB$THREAD_CALLBACKS.get();//先通过 CGLIB$THREAD_CALLBACKS的get（）方法获取拦截对象
        if(temp_2!=null){
            temp_3 = (Callback[])temp_2;
        }
        else if(CGLIB$STATIC_CALLBACKS!=null){  //如果获取不到的话，再从CGLIB$STATIC_CALLBACKS来获取，如果也没有则认为该方法不需要代理。
            temp_3 = CGLIB$STATIC_CALLBACKS;
        }
        else{
            return;
        }
        temp_1.CGLIB$CALLBACK_0 = (MethodInterceptor)temp_3[0];  //设置拦截对象了 
        return；
    }
    
   ///////////////////////////////////////////////////////////////////////
 
```

在CGLIB $SET_THREAD_CALLBACKS方法中调用了CGLIB$THREAD_CALLBACKS的set方法来保存拦截对象，在CGLIB$BIND_CALLBACKS方法中使用了CGLIB$THREAD_CALLBACKS的get方法来获取拦截对象，并保存到CGLIB$CALLBACK_0中。这样，在我们调用代理类的g方法时，就可以获取到我们设置的拦截对象，然后通过  *tmp4_1.intercept(this, CGLIB$g$0$Method, CGLIB$emptyArgs, CGLIB$g$0$Proxy)*  来实现代理。

接下来让我们看看拦截对象中intercept方法：

![选区_743.png](https://i.loli.net/2018/12/26/5c2260493214e.png)



invokeSuper()方法：

![选区_745.png](https://i.loli.net/2018/12/26/5c22619d44bd1.png)

在这里好像不能直接看出代理方法的调用。在JDK动态代理中方法的调用是通过反射来完成的。但是在CGLIB中，方法的调用并不是通过反射来完成的，而是直接对方法进行调用：**FastClass**对Class对象进行特别的处理.

![选区_746.png](https://i.loli.net/2018/12/26/5c2262b61ff93.png)

以我们上面的sayHello方法为例，f1指向委托类对象，f2指向代理类对象，i1和i2分别代表着*sayHello*方法以及*CGLIB$sayHello$0*方法在对象信息数组中的下标。

> 上面代码调用过程就是获取到代理类对应的FastClass，并执行了代理方法。还记得之前生成三个class文件吗？Target$$EnhancerByCGLIB$$f64c12a7$$FastClassByCGLIB$$98e48c91.class就是代理类的FastClass，Target$$FastClassByCGLIB$$d3bf33eb.class就是被代理类的FastClass。



## 3.Fastclass机制分析

   Jdk动态代理的拦截对象是通过反射的机制来调用被拦截方法的，反射的效率比较低，所以cglib采用了FastClass的机制来实现对被拦截方法的调用。FastClass机制就是对一个类的方法建立索引，通过索引来直接调用相应的方法，下面用一个小例子来说明一下，这样比较直观：

```java
class Test{
    public void f(){
        System.out.println("f method");
    }
    
    public void g(){
        System.out.println("g method");
    }
}

//Test2是Test的Fastclass
class Test2{
    public Object invoke(int index, Object o, Object[] ol){
        Test t = (Test) o;
        switch(index){
        case 1:
            t.f();
            return null;
        case 2:
            t.g();
            return null;
        }
        return null;
    }
    
    public int getIndex(String signature){
        switch(signature.hashCode()){
        case 3078479:
            return 1;
        case 3108270:
            return 2;
        }
        return -1;
    }
}

```

在Test2中有两个方法getIndex和invoke。在getIndex方法中对Test的每个方法建立索引，并根据入参（方法名+方法的描述符）来返回相应的索引。Invoke根据指定的索引，以ol为入参调用对象O的方法。这样就避免了反射调用，提高了效率。代理类（Target$$EnhancerByCGLIB$$788444a0）中与生成Fastclass相关的代码如下

## 4.总结

到此为止CGLIB动态代理机制就介绍完了，下面给出三种代理方式之间对比。

| 代理方式      | 实现                                                         | 优点                                                         | 缺点                                                         | 特点                                                       |
| ------------- | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------------------- |
| JDK静态代理   | 代理类与委托类实现同一接口，并且在代理类中需要硬编码接口     | 实现简单，容易理解                                           | 代理类需要硬编码接口，在实际应用中可能会导致重复编码，浪费存储空间并且效率很低 | 好像没啥特点                                               |
| JDK动态代理   | 代理类与委托类实现同一接口，主要是通过代理类实现InvocationHandler并重写invoke方法来进行动态代理的，在invoke方法中将对方法进行增强处理 | 不需要硬编码接口，代码复用率高                               | 只能够代理实现了接口的委托类                                 | 底层使用反射机制进行方法的调用                             |
| CGLIB动态代理 | 代理类将委托类作为自己的父类并为其中的非final委托方法创建两个方法，一个是与委托方法签名相同的方法，它在方法中会通过super调用委托方法；另一个是代理类独有的方法。在代理方法中，它会判断是否存在实现了MethodInterceptor接口的对象，若存在则将调用intercept方法对委托方法进行代理 | 可以在运行时对类或者是接口进行增强操作，且委托类无需实现接口 | 不能对final类以及final方法进行代理                           | 底层将方法全部存入一个数组中，通过数组索引直接进行方法调用 |