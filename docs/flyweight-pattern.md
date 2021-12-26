## 享元模式

享元模式(Flyweight Design Pattern)，顾名思义就是共享单元。享元模式的意图是复用对象，节省内存，前提是享元对象是不可变对象。

具体来讲，当一个系统中存在大量重复对象的时候，如果这些重复的对象是不可变对象，我们就可以利用享元模式将对象设计成享元，在内存中只保留一份实例，供多处代码引用。这样可以减少内存中对象的数量，起到节省内存的目的。实际上，不仅相同对象可以设计成享元，对于相似对象，我们也可以将这些对象中相同的部分（字段）提取出来，设计成享元，让这些大量相似对象引用这些享元。

这里我稍微解释-下，定义中的“不可变对象”指的是，一旦通过构造函数初始化完成之后，它的成员变量或者属性就不会再被修改了。所以，不可变对象不能暴露任何 set() 等修改内部状态的方法。之所以要求享元是不可变对象，那是因为它会被多处代码共享使用，避免一处代码对享元进行了修改，影响到其他使用它的代码。

### 享元模式解析

下面我们通过一个简单的例子解释一下享元模式。假设我们需要使用线条和椭圆形创建图形。因此，我们将有一个接口 Shape 和它的具体实现为 Line 和 Oval。椭圆类将具有固有属性，以确定是否用给定的颜色填充椭圆，而 Line 将不具有任何固有属性。

```java
public interface Shape {

	public void draw(int x, int y, int width, int height, Color color);
}

public class Line implements Shape {

	public Line(){
		System.out.println("Creating Line object");
	}

	@Override
	public void draw(int x1, int y1, int x2, int y2, Color color) {
		System.out.println("Draw Line");
	}

}

public class Oval implements Shape {
	
	// 内部属性
	private boolean fill;
	
	public Oval(boolean isFill){
		this.fill = fisFill;
		System.out.println("Creating Oval object");
	}
	@Override
	public void draw(int x, int y, int width, int height,
			Color color) {
		System.out.println("Draw Oval");
		if(fill){
			System.out.println("Fill Oval with color");
		}
	}
}

public class ShapeFactory {

	private static final Map<ShapeType,Shape> shapes = new HashMap<ShapeType,Shape>();

	public static Shape getShape(ShapeType type) {
		Shape shape = shapes.get(type);

		if (shape == null) {
			if (type.equals(ShapeType.OVAL_FILL)) {
				shape = new Oval(true);
			} else if (type.equals(ShapeType.OVAL_NOFILL)) {
				shape = new Oval(false);
			} else if (type.equals(ShapeType.LINE)) {
				shape = new Line();
			}
			shapes.put(type, shape);
		}
		return shape;
	}
	
	public static enum ShapeType {
		OVAL_FILL, OVAL_NOFILL, LINE;
	}
}

public class Client {

  public static void main(String[] args) {
    Shape shape = ShapeFactory.getShape(ShapeType.LINE);
    // 伪代码
    shape.draw(x1, y1, x2, y2);

    Shape shape1 = ShapeFactory.getShape(ShapeType.LINE);
    // 伪代码
    shape1.draw(x1, y1, x2, y2);
  }
}
```

当我们多次创建 Line 图形时，由于程序使用了共享库，因此程序将快速执行。从这个例子可以看出，实现对象复用最简单的方式是，用一个 HashMap 来存放每次新生成的对象。每次需要一个对象的时候，先到 HashMap 中看看有没有，如果没有，再生成新的对象，然后将这个对象放入 HashMap 中。

### 享元模式 VS 缓存、对象池

在上面的内容中，我们多次提到“共享”、“缓存”、“复用”，那享元模式与缓存、对象池这些概念有何区别呢？下面我们就来看看吧。

**享元模式与缓存区别**

在享元模式的实现中，我们通过工厂类来“缓存”已经创建好的对象。这里的“缓存”实际上是“保存”的意思，跟我们平时所说的“数据库缓存”、“内存缓存”、“Redis 缓存”是两回事。我们平时所讲的缓存，主要是为了提高访问速度，而非复用。

**享元模式与对象池区别**

之前，我在 B 站看到有个设计模式的视频说享元模式的经典应用是池技术（比如对象池、连接池、线程池）。其实这个说法是不严谨的。

虽然对象池、连接池、线程池、享元模式都是为了复用，但是，如果我们再仔细地捋一捋“复用”这个词的话，对象池、连接池、线程池等池技术中的“复用”和享元模式中的“复用”实际上是不同的概念。

池技术中的“复用”我们可以理解为“重复使用”，它主要目的是节省时间（比如从数据库池中获取一个连接，不需要重新创建。我们都知道创建数据库连接是一个很费时的事）。在任何时刻，每一个对象、连接、线程，并不会被多处使用，而是被一个使用者独占，当使用完成之后，又放回到池中，然后再由其他使用者重复利用。享元模式中的“复用”可以理解为“共享使用”，在整个生命周期中，都是被所有使用者共享的，主要目的是节省空间。

### 享元模式的常见应用

刚才我们说到，享元模式的经典应用不是池技术，那享元模式到底用在了哪些地方呢？

**享元模式在 Java 在包装器类型中的应用**

在 Java 中，所有的基本数据类型都有对应的包装器类型：int -> Integer，long -> Long，float -> Float，double -> Double，boolean -> Boolean，short -> Short，byte -> Byte，char -> Character。它们之间可以进行**自动装箱**和**自动拆箱**。

这里补充一下什么是自动装箱和自动拆箱。所谓的自动装箱，就是自动将基本数据类型转换为包装器类型。所谓的自动拆箱，也就是自动将包装器类型转化为基本数据类型。

```java
Integer a = 64；// 自动装箱
int b = a；// 自动拆箱
```

数值 64 是基本数据类型 int，当赋值给包装器类型（Integer）变量的时候，会触发自动装箱操作，创建一个 Integer 类型的对象，并且赋值给变量 a。反过来，当把包装器类型的变量 a，赋值给基本数据类型变量 b 的时候，也会触发自动拆箱操作，将 a 中的数据取出，赋值给 b。其实，它们底层是这样执行的：

```java
Integer a = 59；// 底层执行的是：Integer a = Integer.valueOf(59);
int b = a // 底层执行的是：int b = a.intValue();
```

讲了那么多，也没有提到享元模式是如何在 Java 包装器类中应用的。下面我们就进入正题，先来看看下面这段代码：

```java
Integer a = 64;
Integer b = 64;
Integer c = 129;
Integer d = 129;
System.out.println(a == b);
System.out.println(c == d);
```

前 4 行赋值语句都会触发自动装箱操作，也就是会创建 Integer 对象并且赋值给 a、b、c、d 这四个变量。按照一贯的思维，因为 a、b、c、d 都是对象，所以使用“==”来比较是否相同的时候，比较的是地址，会返回 false。

但是，这样的分析是不对的，答案并非都是 false，而是一个 true，一个 false。看到这里，有些小伙伴就可能有点懵了。其实是因为 Integer 用到了享元模式来复用对象，才导致了这样的运行结果。当进行自动装箱时，也就是调用 valueOf() 来创建 Integer 对象的时候，如果要创建的 Integer 对象的值在 -128 到 127 之间，会从 IntegerCache 类中直接返回，否则才进行 new 创建。**Integer 部分源码如下**：

```java
// valueOf() 方法源码
public static Integer valueOf(int i) {
		if (i >= IntegerCache.low && i <= IntegerCache.high)
				return IntegerCache.cache[i + (-IntegerCache.low)];
		return new Integer(i);
}

/**
* Cache to support the object identity semantics of autoboxing for values between
* -128 and 127 (inclusive) as required by JLS.
*
* The cache is initialized on first usage.  The size of the cache
* may be controlled by the {@code -XX:AutoBoxCacheMax=<size>} option.
* During VM initialization, java.lang.Integer.IntegerCache.high property
* may be set and saved in the private system properties in the
* sun.misc.VM class.
*/
private static class IntegerCache { // IntegerCache 类，是 Integer 的内部类
		static final int low = -128;
		static final int high;
		static final Integer cache[];

		static {
				// high value may be configured by property
				int h = 127;
				String integerCacheHighPropValue =
						sun.misc.VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
				if (integerCacheHighPropValue != null) {
						try {
								int i = parseInt(integerCacheHighPropValue);
								i = Math.max(i, 127);
								// Maximum array size is Integer.MAX_VALUE
								h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
						} catch( NumberFormatException nfe) {
								// If the property cannot be parsed into an int, ignore it.
						}
				}
				high = h;

				cache = new Integer[(high - low) + 1];
				int j = low;
				for(int k = 0; k < cache.length; k++)
						cache[k] = new Integer(j++);

				// range [-128, 127] must be interned (JLS7 5.1.7)
				assert IntegerCache.high >= 127;
		}

		private IntegerCache() {}
}
```
通过上面的源码，我们可以清晰的知道，因为 64 处于 -128 到 127 之间，所以 a 和 b 指向相同的享元对象，所以 a == b 返回 true；而 129 大于 127，并不会被缓存成享元对象，在进行自动包装时会创建一个全新的对象，也就是说，c 和 d 指向不同的对象，所以 c == d 返回 false。

下面小伙们思考一个问题，为什么 IntegerCache 只缓存 -128 到 127 之间的整型值呢？

在 IntegerCache 的代码实现中，当这个类被加载的时候，缓存的享元对象会被集中一次性创建好。毕竟整型值太多了，我们不可能在 IntegerCache 类中预先创建好所有的整型值，这样既占用太多内存，也使得加载 IntegerCache 类的时间过长。所以，我们只能选择缓存对于大部分应用来说最常用的整型值，也就是一个字节的大小（-128到 127之间的数据）。

除了 Ineger 类型之外，Long、Short、Byte 类型也都利用享元模式来缓存 -128 到 127 之间的数据，还有 Character 类型是利用享元模式来缓存 0 到 127 之间的字符。有兴趣的小伙伴，可以阅读一下它们的源码。

**享元模式在 Java String 中的应用**

String 类的设计思路与上面提到的 Integer、Long 等类的设计思路相似，也是利用享元模式来复用相同的字符串常量。JVM 会专门开辟一块存储区来存储字符串常量。这块存储区叫作”字符串常量池“。

```java
String a = "abc";
String b = "abc";
String c = new String("abc");
System.out.println(a == b);
System.out.println(a == c);
```
上面代码运行结果也是：一个 true， 一个 false。

不过，String 类的享元模式的设计，跟 Integer 类稍微有些不同。Integer 类中要共享的对象，是在类加载的时候，就集中一次性创建好的。但是，对于字符串来说，我们没法事先知道要共享哪些字符串常量，所以没办法事先创建好，只能在某个字符串常量第一次被用到的时候，存储到常量池中，当之后再用到的时候，直接引用常量池中已经存在的即可，就不需要再重新创建了。

### 总结

当我们需要创建一个类的许多对象且没有足够的内存容量时，可以使用享元模式。由于每个对象都会占用至关重要的内存空间，因此可以应用享元模式来通过共享对象来减少内存负载，同时也能提高程序效率。

事实上，享元模式可以避免大量非常相似类的开销。在程序设计中，有时需要生成大量细粒度的类实例来表示数据。如果能发现这些实例除了几个参数外基本上都是相同的，有时就能够受大幅度地减少需要实例化的类的数量。如果能把那些参数移到类实例的外面，在方法调用时将它们传递进来，就可以通过共享大幅度地减少单个实例的数目。

**额外补充**：享元模式对 JVM 的垃圾回收并不友好。因为享元工厂类一直保存了对享元对象的引用，这就导致享元对象在没有任何代码使用的情况下，也并不会被 JVM 垃圾回收机制自动回收掉。因此，在某些情况下，如果对象的生命周期很短，也不会被密集使用，利用享元模式反倒可能会浪费更多的内存。所以，除非经过线上验证，利用享元模式真的可以大大节省内存，否则，就不要过度使用这个模式，为了一点点内存的节省而引入一个复杂的设计模式，得不偿失。
