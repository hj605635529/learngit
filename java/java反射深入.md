# java反射深入

[TOC]

##浅析invoke过程

invoke方法用来在运行时动态地调用某个实例的方法。它的实现如下:

```java
@CallerSensitive
public Object invoke(Object obj, Object... args)
    throws IllegalAccessException, IllegalArgumentException,
       InvocationTargetException
{
    if (!override) {
        if (!Reflection.quickCheckMemberAccess(clazz, modifiers)) {
            Class<?> caller = Reflection.getCallerClass();
            checkAccess(caller, clazz, obj, modifiers);
        }
    }
    MethodAccessor ma = methodAccessor;             // read volatile
    if (ma == null) {
        ma = acquireMethodAccessor();
    }
    return ma.invoke(obj, args);
}
```

我们根据invoke方法的实现，将其分为以下几步：

### 权限检查

invoke方法首先会检查AccessibleObject的override属性的值。AccessibleObject 类是 Field、Method 和 Constructor 对象的基类。它提供了将反射的对象标记为在使用时取消默认 Java 语言访问控制检查的能力。

override的值默认是false,表示需要权限调用规则，调用方法时需要检查权限;我们也可以用setAccessible方法设置为true,若override的值为true，表示忽略权限规则，调用方法时无需检查权限（也就是说可以调用任意的private方法，违反了封装）。

如果override属性为默认值false，则进行进一步的权限检查：

（1）首先用Reflection.quickCheckMemberAccess(clazz, modifiers)方法检查方法是否为public，如果是的话跳出本步；如果不是public方法，那么用Reflection.getCallerClass()方法获取调用这个方法的Class对象，这是一个native方法:

```java
@CallerSensitive
public static native Class<?> getCallerClass();
```

获取了这个Class对象caller后用checkAccess方法做一次快速的权限校验，其实现为:

```java
void checkAccess(Class<?> caller, Class<?> clazz, Object obj, int modifiers)
    throws IllegalAccessException
{
    if (caller == clazz) {  // quick check
        return;             // ACCESS IS OK
    }
    Object cache = securityCheckCache;  // read volatile
    Class<?> targetClass = clazz;
    if (obj != null
        && Modifier.isProtected(modifiers)
        && ((targetClass = obj.getClass()) != clazz)) {
        // Must match a 2-list of { caller, targetClass }.
        if (cache instanceof Class[]) {
            Class<?>[] cache2 = (Class<?>[]) cache;
            if (cache2[1] == targetClass &&
                cache2[0] == caller) {
                return;     // ACCESS IS OK
            }
            // (Test cache[1] first since range check for [1]
            // subsumes range check for [0].)
        }
    } else if (cache == caller) {
        // Non-protected case (or obj.class == this.clazz).
        return;             // ACCESS IS OK
    }

    // If no return, fall through to the slow path.
    slowCheckMemberAccess(caller, clazz, obj, modifiers, targetClass);
}
```

首先先执行一次快速校验，一旦调用方法的Class正确则权限检查通过。 若未通过，则创建一个缓存，中间再进行一堆检查（比如检验是否为protected属性）。 如果上面的所有权限检查都未通过，那么将执行更详细的检查，其实现为：

```java
void slowCheckMemberAccess(Class<?> caller, Class<?> clazz, Object obj, int modifiers,
                           Class<?> targetClass)
    throws IllegalAccessException
{
    Reflection.ensureMemberAccess(caller, clazz, obj, modifiers);

    // Success: Update the cache.
    Object cache = ((targetClass == clazz)
                    ? caller
                    : new Class<?>[] { caller, targetClass });

    // Note:  The two cache elements are not volatile,
    // but they are effectively final.  The Java memory model
    // guarantees that the initializing stores for the cache
    // elements will occur before the volatile write.
    securityCheckCache = cache;         // write volatile
}
```

大体意思就是，用Reflection.ensureMemberAccess方法继续检查权限，若检查通过就更新缓存，这样下一次同一个类调用同一个方法时就不用执行权限检查了，这是一种简单的缓存机制。由于JMM的happens-before规则能够保证缓存初始化能够在写缓存之前发生，因此两个cache不需要声明为volatile。 到这里，前期的权限检查工作就结束了。如果没有通过检查则会抛出异常，如果通过了检查则会到下一步。

### 调用MethodAccessor的invoke方法

Method.invoke()实际上并不是自己实现的反射调用逻辑，而是委托给sun.reflect.MethodAccessor来处理。 首先要了解Method对象的基本构成，每个Java方法有且只有一个Method对象作为root，它相当于根对象，对用户不可见。当我们创建Method对象时，我们代码中获得的Method对象都相当于它的副本（或引用）。root对象持有一个MethodAccessor对象，所以所有获取到的Method对象都共享这一个MethodAccessor对象，因此必须保证它在内存中的可见性。root对象其声明及注释为：

```java
public final class Method extends Executable {
    private Class<?>            clazz;
    private int                 slot;
    // This is guaranteed to be interned by the VM in the 1.4
    // reflection implementation
    private String              name;
    private Class<?>            returnType;
    private Class<?>[]          parameterTypes;
    private Class<?>[]          exceptionTypes;
    private int                 modifiers;
    // Generics and annotations support
    private transient String              signature;
    // generic info repository; lazily initialized
    private transient MethodRepository genericInfo;
    private byte[]              annotations;
    private byte[]              parameterAnnotations;
    private byte[]              annotationDefault;
    private volatile MethodAccessor methodAccessor;
    // For sharing of MethodAccessors. This branching structure is
    // currently only two levels deep (i.e., one root Method and
    // potentially many Method objects pointing to it.)
    //
    // If this branching structure would ever contain cycles, deadlocks can
    // occur in annotation code.
    private Method              root;
```

那么MethodAccessor到底是个啥玩意呢？

```java
public interface MethodAccessor {
    Object invoke(Object var1, Object[] var2) throws IllegalArgumentException, InvocationTargetException;
}
```

可以看到MethodAccessor是一个接口，定义了invoke方法。分析其Usage可得它的具体实现类有:

- abstract class MethodAccessorImpl extends MagicAccessorImpl implements MethodAccessor
- class DelegatingMethodAccessorImpl extends MethodAccessorImpl
- class NativeMethodAccessorImpl extends MethodAccessorImpl

第一次调用Method对象的invoke方法之前，实现调用逻辑的MethodAccessor对象还没有创建；第一次调用才创建MethodAccessor并更新给root,然后调用MethodAccesssor.invoke()完成反射调用。

```java
private MethodAccessor acquireMethodAccessor() {
    // First check to see if one has been created yet, and take it
    // if so
    MethodAccessor tmp = null;
    if (root != null) tmp = root.getMethodAccessor();
    if (tmp != null) {
        methodAccessor = tmp;
    } else {
        // Otherwise fabricate one and propagate it up to the root
        tmp = reflectionFactory.newMethodAccessor(this);
        setMethodAccessor(tmp);
    }

    return tmp;
}
```

可以看到methodAccessor实例由reflectionFactory对象操控生成，它在AccessibleObject下的声明如下:

```java
static final ReflectionFactory reflectionFactory =
    AccessController.doPrivileged(
        new sun.reflect.ReflectionFactory.GetReflectionFactoryAction());
```
再研究一下sun.reflect.ReflectionFactory类的源码：

```java
public MethodAccessor newMethodAccessor(Method var1) {
    checkInitted();
    if (noInflation && !ReflectUtil.isVMAnonymousClass(var1.getDeclaringClass())) {
        return (new MethodAccessorGenerator()).generateMethod(var1.getDeclaringClass(), var1.getName(), var1.getParameterTypes(), var1.getReturnType(), var1.getExceptionTypes(), var1.getModifiers());
    } else {
        NativeMethodAccessorImpl var2 = new NativeMethodAccessorImpl(var1);
        DelegatingMethodAccessorImpl var3 = new DelegatingMethodAccessorImpl(var2);
        var2.setParent(var3);
        return var3;
    }
}
```
MethodAccessor实现有两个版本，一个是Java版本，一个是native版本，两者各有特点。初次启动时Method.invoke()和Constructor.newInstance()方法采用native方法要比Java方法快3-4倍，而启动后native方法又要消耗额外的性能而慢于Java方法。也就是说，Java实现的版本在初始化时需要较多时间，但长久来说性能较好；native版本正好相反，启动时相对较快，但运行时间长了之后速度就比不过Java版了。这是HotSpot的优化方式带来的性能特性，同时也是许多虚拟机的共同点：跨越native边界会对优化有阻碍作用，它就像个黑箱一样让虚拟机难以分析也将其内联，于是运行时间长了之后反而是托管版本的代码更快些。

为了尽可能地减少性能损耗，HotSpot JDK采用“inflation”的技巧：让Java方法在被反射调用时，开头若干次使用native版，等反射调用次数超过阈值时则生成一个专用的MethodAccessor实现类，生成其中的invoke()方法的字节码，以后对该Java方法的反射调用就会使用Java版本。 这项优化是从JDK 1.4开始的。

研究ReflectionFactory.newMethodAccessor()生产MethodAccessor对象的逻辑，一开始(native版)会生产NativeMethodAccessorImpl和DelegatingMethodAccessorImpl两个对象。 

DelegatingMethodAccessorImpl的源码如下：

```java
class DelegatingMethodAccessorImpl extends MethodAccessorImpl {
    private MethodAccessorImpl delegate;

    DelegatingMethodAccessorImpl(MethodAccessorImpl var1) {
        this.setDelegate(var1);
    }

    public Object invoke(Object var1, Object[] var2) throws IllegalArgumentException, InvocationTargetException {
        return this.delegate.invoke(var1, var2);
    }

    void setDelegate(MethodAccessorImpl var1) {
        this.delegate = var1;
    }
}
```

它其实是一个中间层，方便在native版与Java版的MethodAccessor之间进行切换。

然后下面就是native版MethodAccessor的Java方面的声明：

```java
class NativeMethodAccessorImpl extends MethodAccessorImpl {
    private final Method method;
    private DelegatingMethodAccessorImpl parent;
    private int numInvocations;

    NativeMethodAccessorImpl(Method var1) {
        this.method = var1;
    }

    public Object invoke(Object var1, Object[] var2) throws IllegalArgumentException, InvocationTargetException {
        if (++this.numInvocations > ReflectionFactory.inflationThreshold() && !ReflectUtil.isVMAnonymousClass(this.method.getDeclaringClass())) {
            MethodAccessorImpl var3 = (MethodAccessorImpl)(new MethodAccessorGenerator()).generateMethod(this.method.getDeclaringClass(), this.method.getName(), this.method.getParameterTypes(), this.method.getReturnType(), this.method.getExceptionTypes(), this.method.getModifiers());
            this.parent.setDelegate(var3);
        }

        return invoke0(this.method, var1, var2);
    }

    void setParent(DelegatingMethodAccessorImpl var1) {
        this.parent = var1;
    }

    private static native Object invoke0(Method var0, Object var1, Object[] var2);
}
```

每次NativeMethodAccessorImpl.invoke()方法被调用时，程序调用计数器都会增加1，看看是否超过阈值；一旦超过，则调用MethodAccessorGenerator.generateMethod()来生成Java版的MethodAccessor的实现类，并且改变DelegatingMethodAccessorImpl所引用的MethodAccessor为Java版。后续经由DelegatingMethodAccessorImpl.invoke()调用到的就是Java版的实现了。 到这里，我们已经追寻到native版的invoke方法在Java一侧声明的最底层 - invoke0了