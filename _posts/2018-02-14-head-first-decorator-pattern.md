---
layout: post
title: "Head First Decorator Pattern"
date: 2018-02-14 05:48:58
image: 'http://res.cloudinary.com/degzyaemb/image/upload/c_scale,w_750/v1517372364/superman.png'
description: 《Head First Design Pattern》中关于装饰者模式的介绍。
category: 'design pattern'
tags:
- design pattern
- decorator pattern
twitter_text:
introduction: 《Head First Design Pattern》中关于装饰者模式的介绍。
---

《Head First Design Pattern》书中关于 *装饰者模式* 是通过一个例子来引入的。
该例子为，若需要你为星巴兹咖啡设计订单系统，应该如何设计？

其中的错误例子将有继承以及组合的形式来实现，就是所有的咖啡类将继承Beverage这个饮料类。
其中这个饮料类中包括了`cost()`和`getDescription()`两个方法*(如下图)*，看起来似乎没有什么问题，
但是当该系统需要扩展（比如像要引入调料（牛奶、豆浆、摩卡、奶泡））的时候，就遇到问题了。
因为通过组合的形式，Beverage类中将会引入所有的调料参数和调料的`has()`和`set()`方法，
将会使整个类变得难以理解以及修改。不仅如此，当客户需要双倍摩卡的时候，这种形式将不能实现。

![placeholder](http://img1.itboth.com/93/26/aeeUna.png)

在遵循 **类应该对扩展开放，对修改关闭** 的设计原则之下，书中引入了 *装饰者模式* 这种设计模式，
其设计思路就是通过将装饰者（该例子是调料）和被装饰者（咖啡）设计为相同的超类型，然后装饰者可以
通过包装装饰者（通过依赖注入引入进类中），然后在再上自己的行为（在咖啡的价格/解释上加上调料的价格/解释）。

> 装饰者模式：动态地将责任附加到对象上，若要扩展功能，装饰者提供了比继承更有弹性的替代方案。

对以上例子的设计可以通过以下代码来理解：

```java
public abstract class Beverage { // 将Beverage设计成抽象类
    String description = "Unknown Beverage";
    public String getDescription() { return description; }
    public String toString() {
        return "Description: " + getDescription() + ", Cost: " + cost();
    }
    public abstract double cost(); // 抽象方法，继承且不是抽象的方法必须实现
}
```

```java
public class Espresso extends Beverage{
    public Espresso() { description = "Espresso"; }
    @Override
    public double cost() { return 1.99; }
    public static void main(String []args) {
        System.out.println(new Espresso().getDescription());
    }
}
```

```java
public class Mocha extends CondimentDecorator {
    private Beverage beverage;
    public Mocha(Beverage b) { beverage = b; }
    @Override
    public String getDescription() {
        return beverage.getDescription() + ", Mocha";
    }

    @Override
    public double cost() { return beverage.cost() + .20; }

    public static void main(String []args) {
        Beverage espresso = new Espresso();
        espresso = new Mocha(espresso);
        espresso = new Mocha(espresso);./
        System.out.println(espresso);
    }
}

```

![placeholder](http://img.mukewang.com/57a215a700010da708410592.png)

从以上代码以及类图可以容易理解将调料和咖啡继承相同的父类，并且通过调料中的构造函数引入咖啡的对象，
然后在咖啡的基础上加入自己的价格和描述。

通过上述的描述可以理解到生活中吃拉面需要添加温泉蛋、竹笋等配料的例子该如何设计。
更重要的是，java.io包中就大量的使用了 *装饰者模式*。下面的类图可以将`FileInputStream、
StringBufferInputStream、ByteArrayInputStream`看成是咖啡（被装饰类），将`FilterInputStream`
看成是调料（装饰类），他们都有相同的超类`InputStream`。而`PushbackInputStream`等就是具体的装饰者，
他们在被装饰类中添加了自己的"能力"。（扩展行为 比如像`LineNumberInputStream`就为其加上了计算行数的能力）

![placeholder](https://obrxbqjbi.qnssl.com/blog/image/desgin-patterns/decorator-04.png)

-----
### Summary
1. 继承属于扩展形式之一，但不见得是达到弹性设计的最佳方式。
2. 除了继承，装饰者模式也可以让我们扩展行为。
3. 装饰者可以在被装饰者的行为前面与/或后面加上自己的行为，甚至将被装饰者的行为整个取代掉，而达到特定的目的。
4. 装饰者会导致设计中出现许多小对象，如果过度使用，会让程序变得复杂。