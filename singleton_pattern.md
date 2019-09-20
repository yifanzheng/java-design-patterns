## 单例模式

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

    /**
     * 私有化构造函数，让外部不能进行实例化
     */
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

/**
 * 懒汉式（双重检查）
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
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }

}
```


