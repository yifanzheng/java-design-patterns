## 适配器模式

适配器模式(Adapter Design Pattern)，将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类可以一起工作。对于这个模式，有一个经常被拿来解释它的例子，就是 USB 转接头充当适配器，把两种不兼容的接口，通过转接变得可以一起工作。

### 适配器模式解析

适配器模式有两种实现方式：对象适配器和类适配器。其中，对象适配器使用组合关系来实现，类适配器使用继承关系来实现。下面我们就来看看具体的代码实现吧。

其中，Target 是客户所期待的接口定义，可以是具体的或抽象的类，也可以是接口。Adaptee 是需要适配的类。Adapter 是将 Adaptee 包装成符合 Target 接口定义的接口。

**对象适配器实现**

```java
public interface Target {
    void request();
}

public class Adaptee {

    public void specificRequest() {
        System.out.println("特殊请求！");
    }
}

public class Adapter implements Target {

    private Adaptee adaptee;

    public Adapter(Adaptee adaptee) {
        this.adaptee = adaptee;
    }

    @Override
    public void request() {
       this.adaptee.specificRequest();
       // 省略代码...
    }
}
```

**类适配器实现**

```java
public interface Target {
    void request();
}

public class Adaptee {

    public void specificRequest() {
        System.out.println("特殊请求！");
    }
}

public class Adapter extends Adaptee implements Target {

    public void request() {
       super.specificRequest();
       // 省略代码...
    }
}
```

针对这两种实现方式，在实际的开发中，到底该如何选择呢？判断的标准主要有两个，一个是 Adaptee 接口的个数，另一个是 Adaptee 和 Target 的契合程度。
- 如果 Adaptee 接口并不多，那两种实现方式都可以。
- 如果 Adaptee 接口很多，而且 Adaptee 和 Target 接口定义大部分都相同，那推荐使用类适配器，因为 Adapter 复用父类 Adaptee 的接口，比起对象适配器的实现方式，Adaptor 的代码量要少一些。
- 如果 Adaptee 接口很多，而且 Adaptee 和 Target 接口定义大部分都不相同，那推荐使用对象适配器，因为组合结构相对于继承更加灵活。

### 适配器模式应用场景总结

一般来说，适配器模式可以看作一种“补偿模式”，用来补救设计上的缺陷。如果在设计初期，我们就能协调规避接口不兼容的问题，那这种模式就没有应用的机会了。

那在实际的开发中，什么情况下使用适配器模式才比较合适呢？下面我们来看看它的应用场景吧。

**1. 封装有缺陷的接口设计**  

如果我们依赖的外部系统在接口设计方面有缺陷（比如包含大量静态方法），引入之后会影响到我们自身代码的可测试性。为了隔离设计上的缺陷，我们希望对外部系统提供的接口进行二次封装，抽象出更好的接口设计，这个时候就可以使用适配器模式了。

比如下面这个例子：

```java
// 这个类来自外部系统的
public class OtherManager {

public static void staticFunction1() {
    // 省略代码...
}
public void function2() {
    // 省略代码... 
}

// 使用适配器模式进行重构
public class Target {
    void function1();
    void function2();
    // ...
}

// 适配器
public class OtherManagerAdapter extends OtherManager implements Target {

    public void function1() {
      super.staticFunction1();
    }
    public void function2() {
      super.fucntion2();
    }

    //...
}
```

**2. 替换依赖的外部系统**

当我们把项目中依赖的一个外部系统替换为另一个外部系统的时候，利用适配器模式，可以减少对代码的改动。具体的代码示例如下所示：

```java
// 外部依赖 A
public class A {
    void functionA();
    // ...
}

public class AImpl implements A {
    public void functionA() {
      // ...
    }
}

// 使用外部依赖 A
public class Demo {
    public A a;

    public Demo(A a) {
      this.a = a;
    }
    // ...
}
Demo d = new Demo(new AImpl());


// 将外部依赖 A 换成外部依赖 B
public class BAdapter implements A {
    public B b;

    public BAdapter(B b) {
      this.b = b;
    }

    public void functionA() {
      // ...
      b.functionb();
    }
}

// 使用 BAdapter 替换 A
Demo d = new Demo(new BAdapter());
```

**3. 统一多个类的接口设计** 

某个功能的实现依赖多个外部系统(或者说类)。通过适配器模式，将它们的接口适配为统一的接口定义，然后我们就可以使用多态的特性来复用代码逻辑。下面我们来看看这样一个例子。

假设我们的系统要对用户输入的文本内容做敏感词过滤，为了提高过滤的效果，我们引入了多款第三方敏感词过滤系统，依次对用户输入的内容进行过滤，过滤掉尽可能多的敏感词。但是，每个系统提供的过滤接口都是不同的，这就意味着我们没法复用一套逻辑来调用各个系统。这个时候，我们就可以使用适配器模式，将所有系统的接口适配为统一的接口定义，这样我们可以复用调用敏感词过滤的代码。

下面我们就使用适配器模式实现，具体代码如下：

```java
// A 敏感词过滤系统提供的接口
public class ASensitiveWordsFilter { 
    // text是原始文本，函数输出用***替换敏感词之后的文本
    public String filterWords(String text) {
        // ...
    }
}

// B 敏感词过滤系统提供的接口
public class BSensitiveWordsFilter { 
    public String filter(String text) {
       // ...
    }
}

// C 敏感词过滤系统提供的接口
public class CSensitiveWordsFilter { 
    public String filter(String text，String mask) {
    // ...
    }
}

// 未使用适配器模式的代码：代码的可测试性、扩展性不好
public class Ri skManagement {
    private ASensitiveWordsFilter aFilter = new ASensitiveWordsFilter();
    private BSensitiveWordsFilter bFilter = new BSensitiveWordsFilter();
    private CSensitiveWordsFilter cFilter = new CSensitiveWordsFilter();

    public String filterSensitiveWords(String text) {
        String maskedText = aFilter.filterWords(text);
        maskedText = bFilter.filter (maskedText);
        maskedText = cFilter.filter (maskedText，"***");

        return maskedText;
    }
}

// 使用适配器模式进行改造
public interface SensitiveWordsFilter { 
    // 统一接口定义
    String filter(String text); 
}

public class ASensitiveWordsFilterAdaptor implements SensitiveWordsFilter {
    private ASensitiveWordsFilter aFilter ; 
    public String filter(String text) {
        String maskedText = aFilter.filterWords(text);
        return maskedText;
    }
}
// ...省略 BSensitiveWordsFilterAdaptor、 CSensitiveWordsFilterAdaptor...

// 扩展性更好，更加符合开闭原则，如果添加一个新的敏感词过滤系统，
// 这个类完全不需要改动；而且基于接口而非实现编程，代码的可测试性更好。
public class SensitiveWordsFilterManagement {
    private List<SensitiveWordsFilter> filters = new ArrayList<>();

    public void addSensi tiveWordsFilter (SensitiveWordsFilter filter) {
        filters. add(filter);
    }

    public String filterSensitiveWords(String text) {
        String maskedText = text;
        for (SensitiveWordsFilter filter : filters) {
            maskedText = filter.filter(maskedText);
        }
        return maskedText;
    }
}
```


### 总结

一般地，使用一个已经存在的类，但如果它的接口，也就是它的方法和要求不相同时，就可以考虑使用适配器模式。

其实，前面讲到的[代理模式](proxy-pattern.md)和适配器模式在结构上很相似，都需要一个具体的实现类的实例，只是它们的目的不一样；代理模式做的是增强或扩展原方法的事，而适配器做的只是适配的事，将一个现有方法适配成所需要的方法，是一种事后补救策略。





