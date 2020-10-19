## 模板方法模式

模板方法模式(Template Method)在定义了执行方法的固定顺序，它可以提供默认实现，该实现对于所有或某些子类可能是通用的；也可以提供抽象的方法，让子类决定具体要做什么。在含有继承结构的代码中，模板方法模式是非常常用的。

模板方法模式比较简单，这里就使用一个简单的示例进行说明。

### 模板方法示例

在模板方法模式中，需要有一个抽象类来定义一些默认方法或需要子类实现的抽象方法，同时需要一个模板方法来固定这些方法的顺序。在不改变方法执行顺序的前提下，由子类来决定这些方法的具体内容。


抽象类：

```java
/**
 * 抽象模板类
 */
public abstract class AbstractTemplate {

    /**
     * 模板方法，定义了通用方法的执行顺序，子类不可以进行重写
     */
    public final void templateMethod() {
        this.common();
        this.apply();
        this.hookMethod();
    }

    /**
     * 子类通用的方法，子类可以根据自身需要进行重写覆盖
     */
    protected void common() {
        System.out.println("子类通用的方法");
    }

    /**
     * 需要子类自己实现的方法
     */
    protected abstract void apply();

    /**
     * 钩子方法，不做任何事情，子类可以视情况决定要不要重写覆盖
     */
    protected void hookMethod() {
    }
}
```
模板方法中调用了三个方法，其中 apply() 是抽象方法，子类必须实现它。其实，模板方法中有多少个抽象方法完全无所谓的，根据实际情况进行设置就好了。这里，我们也可以将上面三个方法都设置为抽象方法，让子类来实现。就如一开始说的那样，模板方法只负责定义方法执行的顺序，由子类来决定这些方法具体做什么。

实现类：

```java
/**
 * 子类实现抽象模板方法类
 */
public class ConcreteTemplate extends AbstractTemplate {

    @Override
    protected void apply() {
        System.out.println("子类实现抽象方法 apply");
    }

    @Override
    protected void hookMethod() {
        System.out.println("使用钩子方法");
    }
}
```

客户端调用：

```java
/**
 * Main
 */
public class Main {
    public static void main(String[] args) {
        AbstractTemplate template = new ConcreteTemplate();
        // 调整模板方法，执行固定的步骤
        template.templateMethod();
    }
}
```

### 小结

模，模板方法模式还是比较简单的，关键是要学会如何运用到自己的代码中。