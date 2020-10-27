## 桥接模式

桥接模式(Bridge Design Pattern)，将抽象部分和它的实现部分分离，使它们都可以独立地变化。  

桥接模式基于类的最小设计原则，通过使用封装、聚合及继承等行为让不同的类承担不同的职责。它的主要特点是把抽象与行为实现分离开来，从而可以保持各部分的独立性以及应对它们的功能扩展。

也就是说，实现系统可能有多种方式分类，每一种分类都有可能变化。桥接模式的核心意图就是把这些分类独立出来，让它们各自独立变化，减少它们之间的耦合。

### 桥接模式应用举例

为了更直观的理解什么是桥接模式，我们来看看这个例子：手机操作问题，对不同品牌手机的不同软件功能进行编程，如通讯录、手机游戏等。

我们先来看最简单、最直接的一种实现方式。代码如下所示：

```java
/**
 * Handset
 */
public abstract class Handset {

    public abstract void run();
}

/**
 * HandsetM
 */
public abstract class HandsetM extends Handset {


}

/**
 * HandsetN
 */
public abstract class HandsetN extends Handset {
    
}

/**
 * HandsetMGame
 */
public class HandsetMGame extends HandsetM {

    @Override
    public void run() {
        System.out.println("运行 M 品牌手机的游戏");
    }
}

/**
 * HandsetNGame
 */
public class HandsetNGame extends HandsetN {

    @Override
    public void run() {
        System.out.println("运行 N 品牌手机的游戏");
    }
}

/**
 * HandsetMMP3
 */
public class HandsetMMP3 extends HandsetM {

    @Override
    public void run() {
        System.out.println("运行 M 品牌手机的 MP3");
    }
}

/**
 * HandsetNMP3
 */
public class HandsetNMP3 extends HandsetN {

    @Override
    public void run() {
        System.out.println("运行 N 品牌手机的 MP3");
    }
}

/**
 * Main
 */
public class Main {

    public static void main(String[] args) {
        HandsetM handsetM = new HandsetMGame();
        handsetM.run();

        HandsetN handsetN = new HandsetNMP3();
        handsetN.run();
    }
}
```

这种方式有一个明显的问题，那就是使用多层次继承，在扩展上存在**类爆炸**的问题。当我们要给每个品牌的手机增加一个新功能时，就要在每个品牌的手机下面增加一个子类，这样增加了代码维护的成本。

事实上，很多情况下使用继承会带来麻烦。对象的继承关系是在编译时就定义好了的，所以无法在运行时改变从父类继承的实现。子类的实现与它的父类有非常紧密的依赖关系，以至于父类实现中的任何变化必然导致其子类发生变化。当需要复用子类时，如果继承下来的实现不适合解决新的问题，则父类必须重写或被其他更适合的类替换。这种依赖关系限制了灵活性并最终限制复用。

针对上面的代码，我们可以将手机的不同功能软件（MP3、Game 等）逻辑剥离出来，形成独立手机软件类(HandsetSoft 相关类)。其中，Handset 类相当于抽象，HandsetSoft 类相当于实现，两者可以独立开发，通过组合关系(也就是桥梁)任意组合在一起。所谓任意组合的意思就是，不同品牌的手机和手机功能软件之间的对应关系，不是在代码中固定写死的，我们可以动态地去指定。

**使用桥接模式改进**

```java
/**
 * HandsetSoft
 */
public interface HandsetSoft {

    void run();
}

/**
 * HandsetGame
 */
public class HandsetGame implements HandsetSoft {

    @Override
    public void run() {
        System.out.println("运行手机游戏");
    }
}

/**
 * HandsetMP3
 */
public class HandsetMP3 implements HandsetSoft {

    @Override
    public void run() {
        System.out.println("运行手机 MP3");
    }
}

/**
 * Handset
 */
public abstract class Handset {

    protected HandsetSoft handsetSoft;

    public Handset(HandsetSoft handsetSoft) {
        this.handsetSoft = handsetSoft;
    }

    public abstract void run();
}

/**
 * HandsetM
 */
public class HandsetM extends Handset {

    public HandsetM(HandsetSoft handsetSoft) {
        super(handsetSoft);
    }

    @Override
    public void run() {
        this.handsetSoft.run();
    }
}

/**
 * HandsetN
 */
public class HandsetN extends Hand'se't {

    public HandsetN(HandsetSoft handsetSoft) {
        super(handsetSoft);
    }

    @Override
    public void run() {
        this.handsetSoft.run();
    }
}

// 可以对不同品牌的手机组合不同的功能，灵活性提高
HandsetM handsetM = new HandsetM(new HandsetGame());
handsetM.run();

HandsetN handsetN = new HandsetN(new HandsetMP3());
handsetN.run();
```

### 总结

桥接模式，实现了抽象和实现部分的分离，达到**解耦**的目的，从而极大的提供了系统的灵活性，让抽象部分和实现部分独立开来，这有助于系统进行分层设计，从而产生更好的结构化系统。对于系统的高层部分，只需要知道抽象部分和实现部分的接口就可以了，其它的部分由具体业务来完成。

桥接模式通过组合关系来替代继承关系，避免继承层次引起的类爆炸，降低系统的管理和维护成本。这类似《Effective Java》一书中提倡的”组合优于继承“设计原则。

桥接模式要求正确识别出系统中两个独立变化的维度(抽象和实现)，因此其使用范围有一定的局限性，即需要有这样的应用场景。









