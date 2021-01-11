## 接口隔离原则（ISP）

接口隔离原则的英文翻译是 "Interface Segregation Principle"，缩写为 ISP。 Robert Martin 是这样定义它的："Clients should not be forced to depend upon interfaces that they do not use。" 直译成中文的话就是：客户端不应该强迫依赖它不需要的接口。其中的“客户端”，可以理解为接口的调用者或者使用者。

实际上，“接口”这个名词可以用在很多场合中。生活中我们可以用它来指插座接口等。在软件开发中，我们既可以把它看作一组抽象的约定，也可以具体指系统与系统之间的 API 接口，还可以特指面向对象编程语言中的接口等。

理解接口隔离原则的关键，就是理解其中的“接口”二字。在这条原则中，我们可以把“接口”理解为下面三种东西:
- 一组 API 接口集合
- 单个 API 接口或函数
- OOP 中的接口概念

下面，我们就按照这三种理解方式来讲解一下这条原则。

### 接口：一组 API 接口集合

这里，我们还是通过一个例子来解释一下。用户系统中提供了一组与用户相关的接口给其他模块使用，比如：注册、登录、获取用户信息，删除用户等。具体代码如下：

```java
public interface UserService {
  boolean registe(String username, String password);
  boolean login(String username, String password);
  UserInfo getByUsername(String username);
  void deleteByUsername();
}
```

比如，我们前台模块只需要 UserService 中的注册、登录和获取用户信息接口，并且删除用户接口只限于后台模块使用。而删除用户是一个很慎重的操作，如果把它放在 UserService 中，那使用到 UserService 的模块，都可以使用这个接口。但是，为了提高系统安全性，防止其他模块误删用户，我们并不想其他模块可以调用删除用户的接口。

这里，最简单的解决方案是按照**接口隔离原则**，调用者不应该强迫依赖它不需要的接口，将删除用户接口单独放到一个类中，然后只提供给后台模块使用。代码如下：

```java
public interface UserService {
  boolean registe(String username, String password);
  boolean login(String username, String password);
  UserInfo getByUsername(String username);
  void deleteByUsername();
}

public interface AdminUserService {
  void deleteByUsername();
}

// 后台模块使用
public class AdminUserImpl implements UserService, AdminUserService {
  // ....省略代码...
}
```

在上面例子中，我们把接口隔离原则中的接口，理解为一组接口集合，它可以是某个微服务接口，也可以是某个类库的接口等等。在设计这些接口的时候，如果部分接口只被部分调用者使用，那我们就应该把这部分接口隔离出来，单独给对应的调用者使用，而不是让其他调用者也依赖这部分不会被用到的接口。

### 接口：单个 API 接口或函数

现在，我们又将接口理解成单个接口或者函数。这样，接口隔离原则就可以理解为：函数的功能设计要单一，不要将不同的功能逻辑放在一个函数中。

比如，我们看看下面这个例子，代码如下：

```java
public class Statistics {
  private Long max;
  private Long min;
  private Long average;
}

public Statistics count(List<Long> dataSet) {
  Statistics statistics = new Statistics();
  // ... 各种计算逻辑 ...
  return statistics;
}
```

上面的 count() 函数中，从某种意义上讲，功能不够单一，包含了很多不同的统计功能，比如计算最大值、最小值、平均值。按照接口隔离原则，应该把 count() 函数划分成更小粒度的函数，每个函数负责一个独立的统计功能。比如下面代码所示：

```java
public Long max(List<Long> dataSet) { }
public Long min(List<Long> dataSet) { }
public Long average(List<Long> dataSet) { }
```

不过，如果项目中，对 Statistics 中那几个信息都需要，那 count() 函数的设计就是合理的。如果项目中，有些需求只是需要 Statistics 中的某些信息，那么在这种情况下，就需要把 count() 函数拆分成上面更细粒度的函数。

到这里，我们可以看到接口隔离原则和单一职责原则有点相似，不过还是稍有点区别。单一职责原则主要是针对模块、类、接口的设计。而接口隔离原则更侧重于接口的设计，同时它提供了一种判断接口是否职责单一的标准：通过调用者如何使用接口来间接判断接口是否单一。如果调用者使用部分接口或接口的部分功能，那接口的设计就不够职责单一。

### 接口：OOP 中的接口概念

我们还可以把“接口”理解为 OOP 中的接口概念，比如 Java 中的 interface。下面还是通过一个例子来解释一下。

假设项目中有 A、B、C 三个功能相似的服务类，供其他模块使用。代码如下：

```java
public class A {
  public void funA() {

  }
}

public class B {
  public void funB() {
    
  }
}

public class C {
  public void funC() {
    
  }
}
```
现在，我们需要实现一个新的需求，需要给 A 和 
B 添加一个新功能 funD()，给 B 和 C 添加另一个功能 funE()。

为了实现新需求，我们对代码进行改造。改造后的代码：

```java
public interface D {
  void funD();
}

public interface E {
  void funE();
}

public class A implements D {
  public void funA() {

  }

  @Override
  public void funD() {

  }
}

public class B implements D, E {

  public void funB() {
    
  }

  @Override
  public void funD() {

  }

  @Override
  public void funE() {

  }
}

public class C implements E {
  public void funC() {
    
  }

  @Override
  public void funE() {

  }
}
```

上面例子中，我们将需要添加的功能设计成了非常单一的接口： D 和 E 接口，而服务类只需依赖它们需要的接口就好了，满足了接口隔离原则。相比设计成一个大而全的接口，更加灵活、易复用，同时在代码实现上避免了做一些无用功（服务类实现了它不需要的接口）。


