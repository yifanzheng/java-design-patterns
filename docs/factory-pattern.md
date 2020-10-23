## 工厂模式

一般情况下，工厂模式(Factory Design Pattern)分为三种更细分的类型：简单工厂、工厂方法和抽象工厂。不过，在 GoF 的《设计模式》一书中，将简单工厂模式看作时工厂方法模式的一种特例。

### 简单工厂模式

简单工厂模式是工厂模式中最简单的一种模式。下面，我们通过一个例子来解释一下什么是简单工厂模式。

下面这段代码中，我们根据运算符（+、-、*、/），选择不同的运算操作，得到运算结果。

```java
public class Calculation {

    private double getResult(String operation, double numA, double numB) {
        Operation operation = null;
        if (Objects.equals(operate, "+")) {
            operation = new OperationAdd();
        } else if (Objects.equals(operate, "-")) {
            operation = new OperationSub();
        } else if (Objects.equals(operate, "*")) {
            operation = new OperationMul();
        } else if (Objects.equals(operate, "/")) {
            operation = new OperationDiv();
        } else {
            throw new InvalidOperationException("Operation not supported: " + operation);
        }
        // 使用运算器计算结果
        double result = operation.getResult(numberA, numberB);
        return result;
    }
}
``` 
上面这段代码，我们还可以进一步改进。为了让代码逻辑更清晰，可读性更好，同时保证类的职责更加单一，我们可以功能独立的代码块封装成函数，并将此函数抽离到一个独立的类中。按照这个思路，我们可以将代码中涉及 Operation 创建的部分封装成独立的函数，并将其抽离到一个单独的类中，而这个类就是简单工厂模式类。改进后的代码如下所示：

```java
public class OperationFactory {

    private OperationFactory() { }

    public static Operation newOperation(String operate) {
        Operation operation = null;
        if (Objects.equals(operate, "+")) {
            operation = new OperationAdd();
        } else if (Objects.equals(operate, "-")) {
            operation = new OperationSub();
        } else if (Objects.equals(operate, "*")) {
            operation = new OperationMul();
        } else if (Objects.equals(operate, "/")) {
            operation = new OperationDiv();
        }

        return operation;
    }
}
```
从上面的方法中，我们可以知道，如果我们需要添加新的运算符操作 operation（比如，%），那势必要改动 OperationFactory 的代码，这样就违背了开放封闭原则。实际上，如果我们能确定不需要频繁地向 OperationFactory 的代码中添加新的 operation，只是偶尔会改动一下，稍微不符合开放封闭原则，也是可以接受的。

其次，上面方法中有一组 if 分支，我们也可以使用一个 Map 先将 operation 对象创建好缓存起来，我们要使用的时候，就直接从 Map 中取出来。这样不仅消除了 if 分支，还达到了缓存的作用。具体代码实现如下所示：

```java
public class OperationFactory {

    private static final Map<String, Operation> operationMap = new HashMap<>();

    static {
        operationMap.put("+", new OperationAdd());
        operationMap.put("-", new OperationSub());
        operationMap.put("*", new OperationMul());
        operationMap.put("/", new OperationDiv());
    }

    private OperationFactory() { }

    public static Operation newOperation(String operate) {
       if (operate == null || operate.isEmpty()) {
          return null;
       }

        return operationMap.get(operate);
    }
}
```

实际上，如果 if 分支不是很多，代码中的 if 分支是完全可以接受的。在实际开发中，可以使用多态来替代 if 分支，这也不是没有缺点，它虽然提高了代码的扩展性，更加符合开放封闭原则，但也增加了类的个数，牺牲了代码的可读性。尽管简单工厂模式的代码实现中，有多处 if 分支，违背开放封闭原则，但权衡扩展性和可读性，这样的代码实现在大多数情况下是可以接受的。

### 工厂方法模式  

上面我们已经提到，可以使用多态来替代 if 分支。而工厂方法模式，恰恰就是利用了多态。具体代码实现如下：

```java
/**
 * IFactory
 */
public interface IFactory {

    Operation newOperation();
}

/**
 * AddFactory 加法工厂类
 */
public class AddFactory implements IFactory {

    @Override
    public Operation newOperation() {
        return new OperationAdd();
    }
}

/**
 * SubFactory 减法工厂类
 */
public class SubFactory implements IFactory {

    @Override
    public Operation newOperation() {
        return new OperationSub();
    }
}

/**
 * MulFactory 乘法工厂类
 */
public class MulFactory implements IFactory {

    @Override
    public Operation newOperation() {
        return new OperationMul();
    }
}

/**
 * DivFactory 除法工厂类
 */
public class DivFactory implements IFactory {

    @Override
    public Operation newOperation() {
        return new OperationDiv();
    }
}

/**
 * OperateType
 */
public enum OperateType {

    ADD, MUL, DIV, SUB
}

/**
 * OperateFactoryMaker
 */
public class OperateFactoryMaker {

    public static IFactory makeFactory(OperateType type) {

        if (Objects.equals(OperateType.ADD, type)) {
            return new AddFactory();
        } else if (Objects.equals(OperateType.SUB, type)) {
            return new SubFactory();
        } else if (Objects.equals(OperateType.MUL, type)) {
            return new MulFactory();
        } else if (Objects.equals(OperateType.DIV, type)) {
            return new DivFactory();
        }

        throw new InvalidOperationException("Operate type not supported");
    }
｝
```
使用工厂方法模式时，当我们新增一种 operation 的时候，只需要新增一个实现了 IFactory 接口的 Factory 类即可。所以，工厂方法模式比起简单工厂模式更加符合开闭原则。

这里出现了 OperateFactoryMaker 类，它其实是 OperationFactory 的工厂类，即工厂的工厂。虽然我们解决了之前的 if 分支，但是这里也引入了新的 if 分支。同时，工厂方法模式出现了很多 Factory 类，增加了代码的复杂性。每当我们要新增加一个 operation 操作时，都要创建对应的工厂类。

### 抽象工厂模式  

抽象工厂模式定义了一个接口用于创建相关或有依赖关系的对象簇，而无需指明且体的类。抽象工厂模式是**将简单工厂模式和工厂模式进行整合**。从设计层面看，抽象工厂模式就是对简单工厂模式的改进(或者称为进一步的抽象)。将工厂抽象成两层，抽象工厂和具体实现的工厂子类。程序员可以根据创建对象类型使用对应的工厂子类。这样将单个的简单工厂类变成了工厂簇更利于代码的维护和扩展。

```java
/**
 * Operation
 */
public interface Operation {

    double getResult(double numberA, double numberB);
}

/**
 * OperationAdd 加法运算
 */
public class OperationAdd implements Operation {

    @Override
    public double getResult(double numberA, double numberB) {
        return numberA + numberB;
    }
}

/**
 * OperationSub 减法运算
 */
public class OperationSub implements Operation {

    @Override
    public double getResult(double numberA, double numberB) {
        return numberA - numberB;
    }
}

/**
 * OperationMul 乘法运算
 */
public class OperationMul implements Operation {

    @Override
    public double getResult(double numberA, double numberB) {
        return numberA * numberB;
    }
}

/**
 * OperationDiv 除法运算
 */
public class OperationDiv implements Operation {

    @Override
    public double getResult(double numberA, double numberB) {
        if (numberB == 0) {
            throw new ArithmeticException("除数不能为 0！");
        }
        return numberA / numberB;
    }
}

/**
 * OperationAccess 利用反射技术实例化对象
 */
public class OperationAccess {

    public static Operation createOperation(Class<? extends Operation> clazz) {
        Operation operation = null;
        try {
            operation = clazz.newInstance();
        } catch (InstantiationException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        }
        return operation;
    }
}
```
一般来说，所有在用简单工厂的地方，都可以考虑用反射技术来去除 switch 或 if，解除分支判断带来的耦合。

### 小结

工厂模式是将实例化对象的代码提取出来，放到一个类中统一管理和维护，达到与主项目的依赖关系的解耦目的，从而提高项目的扩展和维护性，例如，我们可以很容易地更改 Operation 类的实现，而客户端程序并不会意识到这一点。在创建对象实例时，不要直接 new 类，而是把这个 new 类的动作放在一个工厂的方法中并返回。



