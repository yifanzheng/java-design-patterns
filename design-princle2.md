### 接口隔离原则

### 示例

1. 画UML类图  

```java
/**
 * IOperation
 */
public interface IOperation {

    void operation1();
    void operation2();
    void operation3();
    void operation4();
    void operation5();
}

/**
 * OperationA
 */
public class OperationA implements IOperation {

    @Override
    public void operation1() {
        System.out.println("A 的 operation1 方法");
    }

    @Override
    public void operation2() {
        System.out.println("A 的 operation2 方法");
    }

    @Override
    public void operation3() {
        System.out.println("A 的 operation3 方法");
    }

    @Override
    public void operation4() {
        System.out.println("A 的 operation4 方法");
    }

    @Override
    public void operation5() {
        System.out.println("A 的 operation5 方法");
    }
}

/**
 * OperationB
 */
public class OperationB implements IOperation {

    @Override
    public void operation1() {
        System.out.println("B 的 operation1 方法");
    }

    @Override
    public void operation2() {
        System.out.println("B 的 operation2 方法");
    }

    @Override
    public void operation3() {
        System.out.println("B 的 operation3 方法");
    }

    @Override
    public void operation4() {
        System.out.println("B 的 operation4 方法");
    }

    @Override
    public void operation5() {
        System.out.println("B 的 operation5 方法");
    }
}

/**
 * A
 */
public class A {

    public void operation1(IOperation operation) {
        operation.operation1();
    }

    public void operation2(IOperation operation) {
        operation.operation2();
    }

    public void operation3(IOperation operation) {
        operation.operation3();
    }
}

/**
 * B
 */
public class B {

    public void operation1(IOperation operation) {
      operation.operation1();
    }

    public void operation4(IOperation operation) {
        operation.operation4();
    }

    public void operation5(IOperation operation) {
        operation.operation5();
    }

}

``` 

根据 UML 类图可知，类 A 通过接口 IOperation 依赖类 OperationA，类 B 通过接口 IOperation 依赖类 OperationB。如果接口 IOperation 对于类 A 和类 B 来说不是最小接口，那么类 OperationA 和类 OperationB 必须去实现它们不需要的方法。

2. 根据接口隔离原则改进  
将 IOperation 接口拆分成几个独立的接口，类 A 和类 B 分别与它们需要的接口建立依赖关系,也就是采用接口隔离原则。

3. UML 类图
```java
/**
 * IOperation1
 */
public interface IOperation1 {

    void operation1();
}

/**
 * IOperation2
 */
public interface IOperation2 {

    void operation2();
    void operation3();
}

/**
 * IOperation3
 */
public interface IOperation3 {

    void operation4();
    void operation5();
}

/**
 * OperationA
 */
public class OperationA implements IOperation1, IOperation2 {

    @Override
    public void operation1() {
        System.out.println("A 的 operation1 方法");
    }

    @Override
    public void operation2() {
        System.out.println("A 的 operation2 方法");
    }

    @Override
    public void operation3() {
        System.out.println("A 的 operation3 方法");
    }
    
}

/**
 * OperationB
 */
public class OperationB implements IOperation1, IOperation3 {

    @Override
    public void operation1() {
        System.out.println("B 的 operation1 方法");
    }

    @Override
    public void operation4() {
        System.out.println("B 的 operation4 方法");
    }

    @Override
    public void operation5() {
        System.out.println("B 的 operation5 方法");
    }
}

/**
 * A 通过接口 IOperation1，IOperation2 依赖 OperationA 类，  
 *   只实现了 operation1，operation2，operation3 方法
 */
public class A {

    public void operation1(IOperation1 operation) {
        operation.operation1();
    }

    public void operation2(IOperation2 operation) {
        operation.operation2();
    }

    public void operation3(IOperation2 operation) {
        operation.operation3();
    }
}

/**
 * B 通过接口 IOperation1，IOperation3 依赖 OperationB 类，  
 *   只实现了 operation1，operation4，operation5 方法
 */
public class B {

    public void operation1(IOperation1 operation) {
      operation.operation1();
    }

    public void operation4(IOperation3 operation) {
        operation.operation4();
    }

    public void operation5(IOperation3 operation) {
        operation.operation5();
    }

}
```

### 依赖倒置原则  

依赖倒置原则是指：  
1. 高层模块不应该依赖低层模块，两者都应该依赖其抽象。
2. 抽象不应该依赖细节，细节应该依赖抽象。
3. 依赖倒置的中心思想是面向接口编程。
4. 依赖倒置原则是基于这样的设计理念：相对细节的多变性，抽象的东西要稳定的多，故以抽象为基础搭建的架构比以细节为基础搭建的架构要稳定的多。在 Java 中，抽象是指接口或者抽象类，细节就是具体的实现类。
5. 使用接口或抽象类的目的是制定好规范，而不涉及具体的操作，把展现细节的任务交给它们的实现类进行完成。

**依赖倒置的注意事项和细节**

- 低层模块尽量都要有接口或抽象类，或者两者都有，这样程序的稳定性更好。
- 变量的声明尽量是接口或抽象类，这样变量的引用和实际对象间，就存在一个缓冲层，便于程序的扩展和优化。
- 继承时遵循里式替换原则。

### 里式替换原则

**面向对象编程中继承性的思考和说明**  
1. 继承包含这样一层含义：父类中凡是已经实现的方法，实际上是在设定规范，虽然它不强制要求所有继承它的子类必须遵循这些规范，但是如果子类对这些已经实现的方法任意修改，就会对整个继承体系造成破坏。
2. 继承在给程序带来便利的同时，也带来了弊端。使用继承会给程序带来侵入性，程序的可移植性降低，增加了程序间的耦合性；如果一个类被其他类所继承，在修改这个类时，必须考虑到它所有的子类，并且父类修改后，所涉及到子类的功能可能会故障。
3. 编程中在使用继承时，遵循里式替换原则。

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


