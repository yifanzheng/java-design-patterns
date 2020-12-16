## 里式替换原则（LSP）

里式替换原则的英文翻译是：Liskov Substitution Principle，缩写为 LSP。这个原则最早是在 1986 年由 Barbara Liskov 提出，他是这么描述这条原则的：  
> If S is a subtype of T, then objects of type T may be replaced with objects of type S, without breaking the program。

大概意思就是：子类对象能够替换程序中父类对象出现的任何地方，并且保证原来程序的逻辑行为不变及正确性不被破坏。

### 如何理解里式替换原则

上面的解释比较抽象，下面我们通过一个简单的例子来解释一下。代码如下，父类 Thransporter 使用网络进行简单的消息传输。子类 AuthThransporter 继承 Thransporter，并在传输消息时增加了 token 验证。

```java
/**
 * Transporter
 */
public class Transporter {

    public void send(Request request) {
        // 发送消息...
    }
}

/**
 * AuthTransporter
 */
public class AuthTransporter extends Transporter {
    private String token;

    public AuthTransporter(String token) {
      this.token = token;
    }

    public void send(Request request) {
        // 请求中加入 token 验证
        request.setHeader("token", token);
        // 发送消息...
    }
}

public class Message {

    public void doSend(Transporter transporter) {
      // 省略一些代码...
      transporter.send(request);
    }
}

Message message = new Message();
// 里式替换原则
message.doSend(new AuthTransporter(token));
```

上面的代码中，子类 AuthTransporter 的设计完全符合里式替换原则，可以替换父类出现的任何位置，并且使原来代码的逻辑行为不变且正确，同时也没有被破坏。

看到这里，小伙伴可能有这样的疑惑了，刚才的代码设计不就是利用了面向对象的多态特性吗？难道里式替换和多态说的是一回事？从刚才的例子和定义描述来看，里式替换原则跟多态看起来是有点类似，但实际上它们说的是两回事。

多态是面向对象编程的一大特性，也是面向对象编程语言的一种语法，它是一种代码实现的思路。而里式替换是一种设计原则，是用来指导**继承关系中子类该如何设计的，子类的设计要保证在替换父类的时候，不改变原有程序的逻辑以及不破坏原有程序的正确性**。

还是刚才的例子，我们对 AuthTransporter 类中的 send() 函数稍微改造一下，对传入的 token 进行验证。

```java
/**
 * AuthTransporter
 */
public class AuthTransporter extends Transporter {
    private String token;

    public AuthTransporter(String token) {
      this.token = token;
    }

    public void send(Request request) {
      if (StringUtils.isBank(this.token)) {
        throw new AuthException("No token");
      }
      // 请求中加入 token 验证
      request.setHeader("token", token);
      // 发送消息...
    }
}
```
改造之后的代码中，如果使用子类 AuthTransporter 对象传入 message.doSend() 函数就有可能抛出异常，但如果 message.doSend() 传入父类 Transporter 对象，那就不会抛出异常。这里子类替换父类传入 message.doSend() 函数之后，整个程序的逻辑行为就发生了改变。

虽然改造之后的代码仍然可以通过 Java 的多态语法，动态地用子类 AuthTransporter 来替换父类 Transporter，也并不会导致程序编译或者运行报错。但是，从设计思路上来讲，AuthTransporter 的设计就违反了里式替换原则。

### 怎样的代码符合里式替换原则

里式替换原则还有一个更加能落地、更具指导意义的描述，那就是“按照约定设计”。什么意思呢？就是子类在设计的时候，要遵循父类的行为约定。
父类定义了函数的行为约定，那子类可以改变函数的内部实现逻辑，但不能改变函数原有的行为约定。这里的行为约定包括：函数声明要实现的功能；对输入、输出、异常的约定；甚至包括注释中所罗列的任何特殊说明。实际上，定义中父类和子类之间的关系，也可以替换成接口和实现类之间的关系。

为了让小伙伴们更好的理解，下面列举几个反例来解释一下。

**子类违背父类声明要实现的功能** 

父类中提供的 sortOrdersByDate() 订单排序函数，是按照创建日期从大到小来给订单排序的，而子类重写这个 sortOrdersByDate() 订单排序函数之后，是按照订单金额来给订单排序的。那子类的设计就违背里式替换原则。

**子类违背父类对输入、输出、异常的约定**

在父类中，某个函数约定：运行出错的时候返回 null；获取数据为空的时候返回空集合(empty collection)。而子类重载函数之后，实现变了，运行出错返回异常(exception)，获取不到数据返回 null。那子类的设计就违背里式替换原则。

在父类中，某个函数约定，输入数据可以是任意整数，但子类实现的时候，只允许输入数据是正整数，负数就抛出，也就是说，子类对输入的数据的校验比父类更加严格，那子类的设计就违背了里式替换原则。

在父类中，某个函数约定，只会抛出 NotFoundException 异常，那子类的设计实现中只允许抛出 NotFoundException 异常，任何其他异常的抛出，都会导致子类违背里式替换原则。

**子类违背父类注释中所罗列的任何特殊说明**

父类中定义的 getBook() 获取书籍函数的注释是这么写的：“获取没有删除（逻辑删除）的书籍”，而子类重写 getBook() 函数之后，可以获取所有书籍（包括逻辑删除的），那这个子类的设计也是不符合里式替换原则的。

以上便是三种典型的违背里式替换原则的情况。除此之外，判断子类的设计实现是否违背里式替换原则，还有一个小窍门，那就是拿父类的单元测试去验证子类的代码。如果某些单元测试运行失败，就有可能说明，子类的设计实现没有完全地遵守父类的约定，子类有可能违背了里式替换原则。

### 最后

里式替换原则其实就是用来指导我们在继承关系中子类该如何设计的一个原则。当父类定义了函数的约定后，那子类可以改变函数内部的实现逻辑，但不能改变函数原有的约定，即保证在替换父类的时候，不改变程序的逻辑及不破坏原有程序的正确性。







