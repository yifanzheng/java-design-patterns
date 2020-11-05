## 建造者模式

建造者模式(Builder Design Pattern)，也叫生成器模式，是一种对象构建模式。建造者模式可以将一个产品的内部表象和产品的生成过程分割开来，从而可以使一个建造过程生成具有不同的内部表象的产品对象。

大家有没有想过这样一个问题：创建对象时，可以直接使用构造函数或者 JavaBeans 模式（setter 方法），为什么还需要建造者模式呢？

### 为什么需要建造者模式

老规矩，我们先来看看这样一个例子，用一个类表示手机的一些指标：品牌、运行内存、存储容量，还有很多可选域：尺寸、颜色等等，并且这些可选域有自己的默认值。

对于这样的类，我们一向习惯使用重叠构造器模式，也就是使用不同数量的参数重载构造器，将一些可选域使用默认值。

```java
public class Phone {

    private String brand;

    private int memory;

    private int storage;

    private String color = "black";

    private int size = 8;

    public Phone(String brand, int memory, int storage) {
        this.brand = brand;
        this.memory = memory;
        this.storage = storage;
    }

    public Phone(String brand, int memory, int storage, String color) {
        this.brand = brand;
        this.memory = memory;
        this.storage = storage;
        this.color = color;
    }

    public Phone(String brand, int memory, int storage, String color, int size) {
        this.brand = brand;
        this.memory = memory;
        this.storage = storage;
        this.color = color;
        this.size = size;
    }
}
```

当想要创建实例对象时，可以利用参数列表最短的构造器，最多的也就 5 个参数，看起来还不算太糟。但是，如果参数的数量逐渐增加，变成了 8 个、10 个，甚至更多，那构造器的参数列表会变的很长，客户端代码会很难编写，并且难以阅读。在使用构造器时，也很容易搞错参数的顺序，导致很难发现的错误。

```java
// 很长一串参数，可读性差
Phone phone = new Phone("iphone", 8, 32, "white", 9);
```

解决这个问题的办法，我想大家应该也想到了，就是前面提到的 JavaBeans 模式，使用 setter 方法给成员变量进行赋值，代替冗长的构造器。我们直接看如下代码：

```java
public class Phone {

    private String brand;

    private int memory;

    private int storage;

    private String color = "black";

    private int size = 8;

    public Phone() {
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public void setMemory(int memory) {
        this.memory = memory;
    }

    public void setStorage(int storage) {
        this.storage = storage;
    }

    public void setColor(String color) {
        this.color = color;
    }

    public void setSize(int size) {
        this.size = size;
    }

    // 省略 getter 方法...
}
```
这种模式弥补了重叠构造器模式的不足。说白点，就是创建实例很容易，没有了冗长的参数列表，代码读起来也很容易。

```java
Phone phone = new Phone();
phone.setBrand("iphone");
phone.setMemory(8);
phone.setStorage(32);
phone.setColor("white");
phone.setSize(9);
```

至此，我们仍然没有使用到建造者模式，通过 setter 方法进行赋值，就能满足目前的需求。但是，如果需求还存在以下三种情况，那么 JavaBeans 模式就不能满足了。

- 如果我们需要对一些必要的参数进行校验，由于构建过程分到了几个 setter 方法中，并且构建顺序可能不一致，我们无法在 setter 方法中验证参数的有效性。

- 其次，如果有些参数之间存在一定的约束条件，比如，storage 要大于 memory，size 要小于 10，那么 JavaBeans 模式就无法适用了。

- 还有一点不足是，JavaBeans 模式阻止了把类做成不可变的可能，也就是说，对象创建好后，就不能修改内部属性的值，那么我们就不能暴露 setter 方法。

幸运的是，我们还可以使用建造者模式，解决以上所有的问题。我们可以把校验逻辑放到 Builder 类中，先创建建造者，并且通过 setter 方法设置建造者的属性值，然后在使用 build()方法真正创建对象之前，做集中的校验，校验通过之后才会创建对象。除此之外，我们把 Phone 的构造函数使用 private 修饰。这样我们就只能通过建造者来创建 Phone 对象。并且 Phone 没有提供任何 setter 方法，这样我们创建出来的对象就是不可变对象了。

具体代码如下：

```java
public class Phone {

    private final String brand;

    private final int memory;

    private final int storage;

    private final String color;

    private final int size;

    private Phone(Builder builder) {
        this.brand = builder.brand;
        this.memory = builder.memory;
        this.storage = builder.storage;
        this.color = builder.color;
        this.size = builder.size;
    }

    // 省略 getter 方法...

    public static class Builder {

        private String brand;

        private int memory;

        private int storage;

        private String color = "black";

        private int size = 8;

        public Phone build() {
            // 将依赖关系校验、约束条件校验、必填参数校验等放到这里面做
            // brand、memory、storage 必填
            if (brand == null || "".equals(brand)) {
                throw new IllegalArgumentException("brand should not be empty");
            }

            if (memory < 0) {
                throw new IllegalArgumentException("memory should not less than 0");
            }

            if (storage < 0) {
                throw new IllegalArgumentException("storage should not less than 0");
            }

            if (memory > storage) {
                throw new IllegalArgumentException("storage should not less than memory");
            }

            return new Phone(this);
        }

        public Builder setBrand(String brand) {
            if (brand == null || "".equals(brand)) {
                throw new IllegalArgumentException("brand should not be empty");
            }
            this.brand = brand;
            return this;
        }

        public Builder setMemory(int memory) {
            if (memory < 0) {
                throw new IllegalArgumentException("memory should not less than 0");
            }
            this.memory = memory;
            return this;
        }

        public Builder setStorage(int storage) {
            if (storage < 0) {
                throw new IllegalArgumentException("storage should not less than 0");
            }
            this.storage = storage;
            return this;
        }

        public Builder setColor(String color) {
            if (color == null || "".equals(color)) {
                throw new IllegalArgumentException("color should not be empty");
            }
            this.color = color;
            return this;
        }

        public Builder setSize(int size) {
            if (size < 0) {
                throw new IllegalArgumentException("size should not less than 0");
            }
            this.size = size;
            return this;
        }
    }

}

// 调用方法
Phone phone = new Builder()
        .setBrand("iphone")
        .setMemory(8)
        .setStorage(32)
        .setColor("white")
        .setSize(9)
        .build();
```
这样既保证了像重叠构造器模式一样的安全性，也保证了像 JavaBeans 模式那样不错的可读性，同时也保证了必要的校验逻辑。

不过，没有什么是完美的，建造者模式也有自身的不足。为了创建对象，必须先创建它的建造者，增加了一定的开销。虽然创建建造者的开销不是那么明显，但是如果系统十分注重性能的话，可能就成问题了。其次，建造者模式的代码实际上有点重复，Phone 中的属性需要在 Builder 类中重新定义一次。


### 建造者模式 VS 工厂模式

实际上，工厂模式是用来创建不同但是相关类型的对象（继承同一父类或者接口的一组子类），由给定的参数来决定创建哪种类型的对象。建造者模式是用来创建一种类型的多种对象，通过设置不同的可选参数来创建不同的对象。

网上有一个经典的例子很好地解释了两者的区别：

顾客走进一家餐馆点餐，我们利用工厂模式，根据用户不同的选择，来制作不同的食物，比如披萨、汉堡、沙拉。对于披萨来说，用户又有各种配料可以定制，比如奶酪、西红柿、起司，我们通过建造者模式根据用户选择的不同配料来制作披萨。

### 总结

简言之，如果类的构造器中有多个参数，在设计类时，就可以考虑使用建造者模式，特别是很多参数是可选的时候。使用建造者模式的客户端代码更加易于阅读和编写，同时可以在创建对象时动态地调整需要设置的参数，所以也十分灵活。








