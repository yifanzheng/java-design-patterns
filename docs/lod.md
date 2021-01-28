### 迪米特法则（LOD）

迪米特法则的英文翻译是：Law of Demeter，缩写是 LOD。它还有更加达意的名字，叫作最小知识原则，英文翻译为：The Least Knowledge Principle。

关于这个设计原则，我们先来看一下它最原汁原味的英文定义：
>Each unit should have only limited knowledge about other units: only units “closely”related to the current unit. Or: Each unit should only talk to its friends; Don’t talk tostrangers.

我们把它直译成中文，就是下面这个样子：
>每个模块 （unit）只应该了解那些与它关系密切的模块 （units： only units “closely” relatedto the current unit）的有限知识（knowledge）。或者说，每个模块只和自己的朋友“说话”（talk），不和陌生人“说话”（talk）。

其实简单地理解就是，不该有直接依赖关系的类之间，不要有依赖；有依赖关系的类之间，尽量只依赖必要的接口。迪米特法则是希望减少类之间的耦合，让类越独立越好。每个类都应该少了解系统的其他部分。一旦发生变化，需要了解这一变化的类就会比较少。