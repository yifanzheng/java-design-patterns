## 单例模式  

单例设计模式(Singleton Design Pattern) 理解起来很简单。顾名思义，就是一个类只允许创建一个对象或实例，那这个类就是一个单例类，这种设计模式就是单例设计模式（单例模式）。

**单例设计模式八种方式**

- 饿汉式(静态常量)
- 饿汉式(静态代码块)
- 懒汉式(线程不安全)
- 懒汉式(线程安全，同步方法)
- 懒汉式(线程安全，同步代码块)
- 双重检查
- 静态内部类
- 枚举

### 饿汉式（静态常量、静态代码块）

```java
/**
 * 饿汉式（静态变量）
 */
public class Singleton {

    /**
     * 私有化构造函数，让外部不能进行实例化
     */
    private Singleton() {
    }

    private static final Singleton INSTANCE = new Singleton();

    public static Singleton getInstance() {
        return INSTANCE;
    }
}

/**
 * 饿汉式（静态代码块）
 */
public class Singleton {

    private Singleton() {
    }

    private static final Singleton INSTANCE;

    static {
        INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}

```

饿汉方式比较简单，就是在类装载的时候就完成实例化，避免了线程同步问题。不过，在类装载的时候就完成实例化，没有达到懒加载的效果，如果从始至终从未使用过这个实例，则会造成内存的浪费。  
这种方式基于 classloder 机制避免了多线程的同步问题。不过，instance 在类装载时就实例化，但是导致类装载的原因很多种，因此不能确定有其他的方式(或者其他的静态方法)导致类装载，这时候初始化 instance 就没有达到懒加载的效果，故可能造成**内存浪费**。

不过，如果初始化操作比较耗时，那我们最好不要等到真正要使用它的时候，才去进行初始化操作，这样会影响系统的性能。比如，在响应客户端的请求的时候，才进行初始化操作，会导致请求的响应时间过长，影响系统使用。对于这种情况，采用饿汉方式是比较可取的。

### 懒汉式（线程不安全）

```java

/**
 * 懒汉式（线程不安全）
 */
public class Singleton {

    /**
     * 私有化构造函数，让外部不能进行实例化
     */
    private Singleton() {
    }

    private static Singleton instance;

    /**
     * 当调用到该方法时，才去创建实例
     */
    public static Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

}

```

此方式虽然起到了懒加载的效果，但是只能在单线程下使用。如果在多线程下，一个线程进入了 `if (instance == null)` 判断语句，还没有来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例，所以在多线程环境下不可使用这种方式。在实际开发中，也不要使用这这种方式。

### 懒汉式（线程安全-同步方法）

```java
/**
 * 懒汉式（线程安全， 同步方法）
 */
public class Singleton {

    /**
     * 私有化构造函数，让外部不能进行实例化
     */
    private Singleton() {
    }

    private static Singleton instance;

    /**
     * 在方法上加入同步处理 synchronized 关键字，解决线程安全问题
     */
    public static synchronized Singleton getInstance() {
        if (instance == null) {
            instance = new Singleton();
        }
        return instance;
    }

}
```

此懒汉方式虽然解决了线程安全问题，但缺点也很明显，因为在 getInstance() 方法上加了锁，导致这个方法的并发度很低。当多个线程调用这个方法时，每个线程都要等到前面的那个线程调用完成后才能调用。如果 getInstance() 被频繁地调用，那么频繁加锁、释放锁以及并发度低等问题，会导致性能问题。在实际开发中，不推荐使用这种方式。

### 懒汉式（线程安全， 同步代码块）

```java

/**
 * 懒汉式（线程安全， 同步代码块）
 */
public class Singleton {

    /**
     * 私有化构造函数，让外部不能进行实例化
     */
    private Singleton() {
    }

    private volatile static Singleton instance;

    /**
     * 加入同步处理 synchronized 关键字，解决线程安全问题
     */
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                instance = new Singleton();
            }
        }
        return instance;
    }
}
```

此方式是对懒汉式（同步方法）的单例模式进行改进，解决效率低的问题，但是这种同步代码块的方式并不能起到线程同步的作用。假如一个线程进入了 `if (instance == nul)` 判断语句块，还未来得及往下执行，另一个线程也通过了这个判断语句，这时便会产生多个实例。

### 懒汉式（双重检查）

```java
/**
 * 懒汉式（双重检查）
 */
public class Singleton {

    /**
     * 私有化构造函数，让外部不能进行实例化
     */
    private Singleton() {
    }

    /**
     * 使用 volatile 修饰，防止实例化时进行指令重排序
     */
    private volatile static Singleton instance;

    /**
     * 加入同步处理 synchronized 关键字，解决线程安全问题
     */
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

}
```

双重检查概念是多线程开发中常使用到的，此段代码中进行了两次 `if (instance== null)` 检查，这样就可以保证线程安全了。同时，实例化代码只用执行一次，后面再次访问时，判断 `if (instance== nll)`，直接返回实例化对象，也不会在进入加锁逻辑中。所以这种方式既解决了线程安全问题，也解决了懒加载问题，同时保证了效率（高并发）。

### 静态内部类方式

静态内部类在它的外部类被装载时并不会被装载，只有在被调用时才会被装载，而且只会被装载一次，在装载时线程是安全的。它既保证了懒加载，也保证了线程安全，所以静态内部类也是一种实现单例模式很好的方式。

```java
public class Singleton {

    private Singleton() {
    }

    private static class SingletonInstance {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonInstance.INSTANCE;
    }
}
```

SingletonInstance 是一个静态内部类，当外部类 Singleton 被加载的时候，并不会创建 SingletonInstance 实例对象。只有当调用 getInstance() 方法时，SingletonInstance 才会被加载，这个时候才会创建 INSTANCE。静态内部类的静态属性只会在第一次加载类的时候初始化，并且 JVM 保证了线程的安全性。所以，这种实现方式既保证了线程安全，又能做到懒加载。

### 枚举

```java

public enum  Singleton1 {
    // 一个属性，保证是单例
    INSTANCE

}
```

使用枚举实现单例模式，不仅可以避免多线程问题，而且还能防止反序列化重新创建对象。这种方式是《Effective Java》作者提倡的方式。

### 总结

单例模式保证了系统内存中该类只存在一个对象，节省了系统资源，对于一些需要频繁创建和销毁的对象，使用单例模式可以提高系统性能。当需要实例化一个单例类的时候，必须要记住使用相应的获取对象的方法，而不是使用 new 的方式。 

**单例模式使用的场景**：需要频繁的进行创建和销毁的对象、创建对象时耗时过多或耗费资源过多(即重量级对象)，但又经常用到的对象、工具类对象、频繁访问数据库或文件的对象（比如数据源、session 工厂等）。

**单例模式缺点**：

- 单例对OOP特性的支持不友好
- 单例会隐藏类之间的依赖关系
- 单例对代码的扩展性不友好
- 单例对代码的可测试性不友好
- 单例不支持有参数的构造函数






