---
layout: post
title: "Thinking in java Inner Classes"
date: 2018-02-18 08:03:01
image: 'http://res.cloudinary.com/degzyaemb/image/upload/c_scale,w_750/v1518871929/The_Imitation_game2_oaqxcx.png'
description: 《Thinking in java》中关于内部类的的总结。
category: 'java'
tags:
- java
- inner classes
twitter_text:
introduction: 《Thinking in java》中关于内部类的的总结。
---

内部类不仅仅只是将代码隐藏起来这么简单，下面将重点介绍匿名内部类和内部类与控制框架的关系。

### 1. 匿名内部类

匿名内部类是没有名字的内部类，由于其只能使用一次，所以通常用来简化代码。

> 而匿名内部类这种奇怪的语法指的是："创建一个继承/实现的匿名类的对象"。
通过new表达式返回的引用被自动向上转型成为父类的引用。

下面将看到一个通过匿名内部类来继承其接口的代码逻辑。

```java
class Wrapping {
    private int i;
    public Wrapping(int i) { this.i = i; }
    public int value() { return i; }
}

public class Parcel8 {
    public Wrapping wrapping(int x) {
        return new Wrapping(x) {
            public int value() {
                return super.value() * 47;
            }
        };
    }

    public static void main(String []args) {
        Parcel8 p = new Parcel8();
        System.out.println(p.wrapping(10).value());
    }
}

```


匿名内部类也可用于简化工厂模式，下面将通过一组工厂模式来解释其原理。

先定义一组接口，包括产品接口和工厂接口。
```java

interface Service {
    void method1();
    void method2();
}

interface ServiceFactory {
    Service getService();
}

```

下面是一组普通的工厂模式的代码
```java
class Implements3 implements Service {
    public void method1() {
        print("Implements3 method1");
    }
    public void method2() {
        print("Implements4 method2");
    }
}

class Implement3Factory implements ServiceFactory {
    public Service getService() {
        return new Implements3();
    }
}

class Implements4 implements Service {
    public void method1() {
        print("Implements3 method1");
    }
    public void method2() {
        print("Implements4 method2");
    }
}

class Implement4Factory implements ServiceFactory {
    public Service getService() {
        return new Implements4();
    }
}

public class Factories2 {
    public static void serviceCustomer(ServiceFactory fac) {
        Service service = fac.getService();
        service.method1();
        service.method2();
    }
    public static void main(String []args) {
        serviceCustomer(new Implement3Factory());
        serviceCustomer(new Implement4Factory());
    }
}

```

接着是一组通过匿名内部类简化后的代码。
```java
class Implements1 implements Service {
    public void method1() {
        print("Implements1 method1");
    }
    public void method2() {
        print("Implements1 method2");
    }
    public static ServiceFactory factory = new ServiceFactory() {
        public Service getService() {
            return new Implements1();
        }
    };
}

class Implements2 implements Service {
    public void method1() {
        print("Implements2 method1");
    }
    public void method2() {
        print("Implements2 method2");
    }
    public static ServiceFactory factory = new ServiceFactory() {
        public Service getService() {
            return new Implements2();
        }
    };
}

public class Factories1 {
    public static void serviceCustomer(ServiceFactory fac) {
        Service service = fac.getService();
        service.method1();
        service.method2();
    }

    public static void main(String []args) {
        serviceCustomer(Implements1.factory);
        serviceCustomer(Implements2.factory);
    }
}

```

仔细看可以发现简化后的代码通过将工厂类放进产品类中，通常来说，一组产品来自一个工厂类，
所以此做法使得简化后的代码更加的清晰易懂。

### 2. 内部类与控制框架

> **控制框架(control framework)**是一类特殊的应用程序框架，它用来解决响应事件的需求。
主要用来响应事件的系统被称作事件驱动系统。应用程序设计中常见的问题之一就是图形用户接口（GUI），
他几乎完全是时间驱动的系统，其使用了大量的内部类。

其实通过内部类实现的控制框架完全就是前几章所介绍的**观察者模式**，其中不同的是，
它遵循"使变化的事物与不变的事物相互分析"的原则，
将Event和Listener封装了起来，利用内部类来实现其中的逻辑构建。

下面将介绍通过其内部类构建好的控制温室运作的系统，其就是控制框架的一个特定实现。

```java
public abstract class Event {
    private long eventTime; // 继承的子类不需要操作该参数，所以设计成private
    protected final long delayTime; // 继承的子类需要使用
    public Event(long delayTime) {
        this.delayTime = delayTime;
        start();
    }
    public void start() { // 当运行start()了才开始计时
        eventTime = System.nanoTime() + delayTime;
    }
    public boolean ready() {
        return System.nanoTime() >= eventTime;
    }
    public abstract void action();
}

```

```java
public class Controller {
    private List<Event> eventList = new ArrayList<>();
    public void addEvent(Event c) { eventList.add(c); }
    public void run() {
        while (eventList.size() > 0) {
            for (Event e : new ArrayList<>(eventList))
                if (e.ready()) {
                    System.out.println(e);
                    e.action();
                    eventList.remove(e);
                }
        }
    }
}

```

```java
/**
 * 控制温室的运作：控制灯光、水、温度调节器的开关，
 * 以及响铃和重新启动系统
 */
public class GreenhouseControllers extends Controller {
    private boolean light = false;
    public class LightOn extends Event {
        // 这里如果不写构造函数就会出错，因为父类中有个参数为final形式，必须要实例化
        public LightOn(long delayTime) { super(delayTime); }
        public void action() { light = true; }
        public String toString() { return "Light is on"; }
    }
    public class LightOff extends Event {
        public LightOff(long delayTime) { super(delayTime); }
        public void action() { light = false; }
        public String toString() { return "Light is off"; }
    }

    private boolean water = false;
    public class WaterOn extends Event {
        public WaterOn(long delayTime) { super(delayTime); }
        public void action() { water = true; }
        public String toString() { return "Water is on"; }
    }
    public class WaterOff extends Event {
        public WaterOff(long delayTime) { super(delayTime); }
        public void action() { water = false; }
        public String toString() { return "Water is off"; }
    }

    // 恒温器
    private String thermostat = "Day";
    public class ThermostatNight extends Event {
        public ThermostatNight(long delayTime) { super(delayTime); }
        public void action() { thermostat = "Night"; }
        public String toString() { return "Thermostat on night setting"; }
    }
    public class ThermostatDay extends Event {
        public ThermostatDay(long delayTime) { super(delayTime); }
        public void action() { thermostat = "Day"; }
        public String toString() { return "Thermostat on day setting"; }
    }

    public class Bell extends Event {
        public Bell(long delayTime) { super(delayTime); }
        public void action() { addEvent(new Bell(delayTime)); } // 运行结束之后会将其从event列表中删除，这时再添加进去
        public String toString() { return "Bing!"; }
    }
    // 不是重启的意思 是重新开始运行传入的event列表的意思
    public class Restart extends Event {
        private Event[] eventList;
        public Restart(long delayTime, Event[] eventList) {
            super(delayTime);
            this.eventList = eventList;
            for (Event e: eventList)
                addEvent(e);
        }
        public void action() {
            for (Event e: eventList) {
                e.start();
                addEvent(e);
            }
            start();
            addEvent(this);
        }
        public String toString() { return "Restarting system"; }
    }
    public static class Terminate extends Event {
        public Terminate(long delayTime) { super(delayTime); }
        public void action() { System.exit(0); }
        public String toString() { return "Terminating"; }
    }
}

```

这是一个很好的例子，并且其中的架构也非常的优美，是可以学习的样式。


Q: 嵌套类和内部类有什么关系？
A: 内部类其实就是将类写在另一个类的内部，所以称为内部类，而当你不需要不需要内部类对象与其外围类对象之间有联系，
那么可以将内部类声明为static。这通常称为**嵌套类。**
正常情况下，不能再接口内部防止任何代码，但嵌套类可以作为接口的一部分。
任何放入接口中的任何类都自动地是public和static的。

Q: 为什么需要内部类？<br>
A: 每个内部类都能独立的继承自一个（接口的）实现，所以无论外围类是否已经继承了某个（接口的）实现，
对于内部类都没有影响。 **换句话说就是，内部类有效的实现了"多重继承"**。

下面是一个实现*"多重继承"*的例子：

```java
class D {}
abstract class E {}
class Z extends D {
    E makeE() { return new E() {}; }
}

public class MultiImplementation {
    static void takesD(D d) {}
    static void takesE(E e) {}
    public static void main(String []args) {
        Z z = new Z();
        takesD(z);
        takesE(z.makeE());
    }
}
```


-----
### Summary
1. 内部类可以自由的访问外围的字段，无需限定条件或特殊许可。
2. 匿名内部类可隐式的继承/实现接口来简化代码。
3. 内部类应用广泛，其中应用程序框架使用非常频繁。要读懂框架代码需要好好理解内部类的原理和结构。