## 代理模式

代理模式（Proxy Design Pattern）在不改变原始类代码的情况下，通过引入代理类来给原始类增加功能。说白了就是可以进行做 **方法包装** 或 **方法增强**。

代理模式主要有三种形式，分别是**静态代理**、**动态代理**、**CGlib 代理**。

### 代理模式解析

下面我们通过一个简单的例子来解释一下代理模式。假如，我们要记录老师授课的时长，在业务代码中我们可以这样做：

```java
public class TeachService {

    public void doTeach() {
       Long startTimestamp = System.currentTimeMillis(0);
       // 省略 teach 业务逻辑...
       System.out.println("Do teach.");

       Long endTimeStamp = System.currentTimeMillis();
       Long teachTime = endTimeStamp - startTimestamp;
       System.out.println("Time: " + teachTime);
    }
}
```

从上面的代码中，我们可以很明显地发现这两个问题：首先，计时相关的代码侵入到了业务代码中，跟业务代码高度耦合。如果我们后面需要清除这部分代码或者需要换成其他的功能，那改动成本比较大。其次，违反了单一职责原则。计时相关的代码与业务代码无关，本不应该放在一个类中。

下面我们分别使用上面三种代理模式，对程序进行改进，看看这三种类型的代理模式的区别。

### 静态代理

若代理类在程序运行前就已经存在，那么这种代理方式被成为**静态代理** ，这种情况下的代理类通常都是我们在 Java 代码中定义的。使用静态代理实现时，需要定义接口或者父类，被代理类（即原始类）与代理类一起实现相同接口或者父类。


这里代理类 TeachProxy 和原始类 TeachImpl 实现相同的接口 Tech。 TeachImpl 类只负责业务功能。代理类 TeachProxy 负责在业务代码执行前后附加其他逻辑代码，并通过委托的方式调用原始类来执行业务代码。具体的代码如下所示:

```java
/**
 * Teach
 */
public interface Teach {
    void doTeach();
}

/**
 * TeachImpl
 */
public class TeachImpl implements Teach {

    @Override
    public void doTeach() {
        System.out.println("Do teach.");
        // 省略 teach 业务逻辑...
    }
}

/**
 * TeachProxy
 */
public class TeachProxy implements Teach {

    private Teach teach;

    public TeachProxy(Teach teach) {
        this.teach = teach;
    }

    @Override
    public void doTeach() {
       Long startTimestamp = System.currentTimeMillis(0);
       // 委托原始类执行业务功能
       this.teach.doTeach();

       Long endTimeStamp = System.currentTimeMillis();
       Long teachTime = endTimeStamp - startTimestamp;
       System.out.println("Time: " + teachTime);
    }
}

// TeachProxy 使用举例
Teach teachProxy = new TeachProxy(new TeachImpl());
teachProxy.doTeach();
```

**静态代理特点**

在不改变原始类功能的前提下，能通过代理类对原始类进行功能扩展，同时可以实现客户端与原始类间的解耦。

但是，静态代理的局限在于运行前必须编写好代理对象，由于代理类和原始类都要实现相同的接口，所以会有很多代理类，一旦接口增加功能，原始类和代理类都需要进行维护，增加了代码维护成本。

### 动态代理

对于使用静态代理产生的问题，我们可以使用动态代理来解决。

所谓**动态代理**，就是代理类在程序运行时才进行创建。这种情况下，代理类并不是在 Java 代码中定义的，而是在运行时根据我们在 Java 代码中的“指示”动态生成的，然后在系统中使用代理类替换原始类。

使用动态代理实现时，代理类不需要实现接口，原始类需要实现接口；代理对象的生成是利用 JDK 的 API(java.lang.reflect 包)，动态的在内存中创建对象，所以动态代理又叫 JDK 代理。

JDK 实现动态代理，需要使用 `java.lang.reflect` 包下 Proxy 类中的方法：  
`public static Object newProxyInstance(ClassLoader loader, Class<?>[] interfaces, InvocationHandler h)`

方法参数介绍：  
`ClassLoader loader`: 指当前目标对象的类加载器。  
`Class<?>[] interfaces`: 目标对象实现的接口类型。  
`InvocationHandler h`: 处理程序，当执行目标对象的方法时，会触发处理程序，会把当前执行的目标对象的方法作为参数传入。


下面我们使用动态代理进行对程序进行改进，具体代码如下：

```java
/**
 * Teach
 */
public interface Teach {
    void doTeach();
}

/**
 * TeachImpl
 */
public class TeachImpl implements Teach {

    @Override
    public void doTeach() {
        System.out.println("Do teach.");
        // 省略 teach 业务逻辑...
    }
}

/**
 * TeachProxy
 */
public class TeachProxy {

    private Object target;

    public TeachProxyFactory(Object target) {
        this.target = target;
    }

    public Object getInstance() {

        return Proxy.newProxyInstance(
                target.getClass().getClassLoader(),
                target.getClass().getInterfaces(),
                new InvocationHandler() {
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        Long startTimestamp = System.currentTimeMillis(0);
                        Object invoke = method.invoke(target, args);

                        Long endTimeStamp = System.currentTimeMillis();
                        Long teachTime = endTimeStamp - startTimestamp;
                        System.out.println("Time: " + teachTime);
                        return invoke;
                    }
                }
        );
    }
}

// TeachProxy 使用举例
Teach teachProxy = (Teach) new TeachProxy(new TeachImpl()).getInstance();
teachProxy.doTeach();
```
**动态代理特点**

与静态代理相比，动态代理的优势在于可以很方便的对代理类的函数进行统一的处理，而不用修改每个代理类的函数。但是，动态代理的代理类字节码在创建时，需要实现原始类所实现的接口作为参数。如果原始类是没有实现接口而是直接定义业务方法的话，就无法使用 JDK 动态代理了。   

动态代理是基于接口实现的，只能对接口进行代理，不能对类进行代理。

### CGlib 代理

如果原始类并没有定义接口，并且原始类代码并不是我们开发维护的(比如它来自一个第三方的类库)，我们也没办法直接修改原始类，给它重新定义一个接口。在这种情况下，我们就可以使用 CGlib 代理。

CGlib 代理模式是基于继承被代理类生成代理子类，不用实现接口。只需要被代理类是非 final 类即可。(CGlib 代理底层是借助 ASM 字节码技术）它也是一种动态代理。

下面我们来看看具体代码是如何实现的：

```java
/**
 * Teach
 */
public class Teach {

    public void doTeach() {
        System.out.println("Do teach.");
    }
}

/**
 * TeachMethodInterceptor
 */
public class TeachMethodInterceptor implements MethodInterceptor {

    private Class<?> target;

    public TeachMethodInterceptor(Class<?> target) {
        this.target = target;
    }

    public Object getProxyInstance() {
        // 为代理类指定需要代理的类，也即是父类
        Enhancer enhancer = new Enhancer();
        // 设置方法拦截器回调引用，对于代理类上所有方法的调用，
        // 都会调用 CallBack，而 Callback 则需要实现 intercept() 方法进行拦截
        enhancer.setSuperclass(target);
        // 获取动态代理类对象并返回
        enhancer.setCallback(this);
        return enhancer.create();
    }

    @Override
    public Object intercept(Object obj, Method method, Object[] args, MethodProxy proxy) throws Throwable {
        Long startTimestamp = System.currentTimeMillis(0);
        Object result = proxy.invokeSuper(obj, args);

        Long endTimeStamp = System.currentTimeMillis();
        Long teachTime = endTimeStamp - startTimestamp;
        System.out.println("Time: " + teachTime);
        return result;
    }
}

// 使用举例
TeachMethodInterceptor methodInterceptor = new TeachMethodInterceptor(Teach.class);
Teach proxyInstance = (Teach) methodInterceptor.getProxyInstance();
proxyInstance.doTeach();
```

上述代码中，我们通过 CGlib 的 Enhancer 类来指定要代理的原始对象，最终通过调用 `create()` 方法得到代理对象，对这个对象所有非 final 方法的调用都会转发给 `MethodInterceptor.intercept()` 方法，在 `intercept()` 方法里我们可以加入任何逻辑，比如修改方法参数，加入日志功能、安全检查功能等；通过调用 `MethodProxy.invokeSuper()` 方法，我们将调用转发给原始对象，也就是示例中 Teach 类的具体方法。

**CGlib 代理特点**

使用 CGLiB 实现动态代理，CGLib 底层采用 ASM 字节码生成框架，使用字节码技术生成代理类，比使用 Java 反射效率要高。唯一需要注意的是，CGLib 不能对声明为 final 的方法进行代理， 因为 CGLib 原理是动态生成被代理类的子类，是基于继承实现的。

### 代理模式的应用场景

代理模式常用在业务系统中开发一些非功能性需求，比如：监控、统计、权限控制、限流、事务、日志。我们将这些附加功能与业务功能解耦，放到代理类中统一处理，让程序员只需要关注业务方面的开发。除此之外，代理模式还可以用在 RPC框架、缓存等应用场景中。

### 总结

代理模式主要用于进行方法增强，在不改变原始功能的基础上，拓展附加功能，达到原始功能与附加功能解耦的目的。