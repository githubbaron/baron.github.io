---
layout: post
title: "Head First Observer Pattern"
date: 2018-02-13 06:47:43
image: 'http://res.cloudinary.com/degzyaemb/image/upload/c_scale,w_750/v1517372365/Downton_Abbey.png'
description: 《Head First Design Pattern》中关于观察者模式的介绍。
category: 'design pattern'
tags:
- design pattern
- observer pattern
twitter_text:
introduction: 《Head First Design Pattern》中关于观察者模式的介绍。
---

### 解释观察者模式

> 观察者模式：观察者模式是软体设计模式的一种。在此种模式中，一个目标物件管理所有相依于它的观察者物件，
并且在它本身的状态改变时主动发出通知。这通常透过呼叫各观察者所提供的方法来实现。
此种模式通常被用来实时事件处理系统。 ----Wikipedia

观察者类图：
![placeholder](http://design-patterns.readthedocs.io/zh_CN/latest/_images/Obeserver.jpg)

通过定义和类图可能还是没有办法非常好的理解其中的意思，其实观察者模式就是通过一个对象的变化触发另一个对象
的事件从而使其起到"被通知"的效果，而一个监听能够触发多个事件就是通过在监听中放置collections集合来存储
需要触发的事件对象，当产生变化时，则通过遍历这些对象逐个进行事件触发。

下面将通过《Head First Design Pattern》中的一个天气预报的例子来解释其原理。

```java
public interface Subject { // 监听者
    void registerObserver(MyObserver o);
    void removeObserver(MyObserver o);
    void notifyObserver();
}

public class WeatherData implements Subject {
    private ArrayList<MyObserver> observers = new ArrayList<>();
    private float temperature;
    private float humidity;
    private float pressure;
    public void registerObserver(MyObserver o) { observers.add(o); }

    public void removeObserver(MyObserver o) { observers.remove(o); }

    public void notifyObserver() {
        for (int i = 0; i < observers.size(); i++)
            observers.get(i).update(temperature, humidity, pressure);
    }

    public void measurementsChanged() { notifyObserver(); }

    public void setMeasurements(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        this.pressure = pressure;
        measurementsChanged();
    }
}

```

上面是一个简单监听者的例子，实现监听者接口（就是必须实现其中的`addObserver()`等方法），
并当监听者参数变化时，通知`notifyObserver()`方法来对每一个observer对象进行事件的调用(`update方法`)。

```java
public interface DisplayElement {
    void display();
}

public interface MyObserver {
    void update(float temp, float humidity, float pressure);
}

public class CurrentConditionsDisplay implements MyObserver, DisplayElement{
    private float temperature;
    private float humidity;
    private Subject weatherData;
    public CurrentConditionsDisplay(Subject subject) {
        weatherData = subject;
        weatherData.registerObserver(this);
    }
    public void display() { System.out.println("Current conditions: " + temperature + "F degrees and " + humidity + "% humidity"); }

    public void update(float temperature, float humidity, float pressure) {
        this.temperature = temperature;
        this.humidity = humidity;
        display();
    }

    public static void main(String []args) {
        WeatherData subject = new WeatherData();
        CurrentConditionsDisplay ccd = new CurrentConditionsDisplay(subject);
        subject.setMeasurements(34, 23, 12);
    }
}

```

事件对象则通过初始化阶段传入监听者对象来将事件对象添加进事件collections列表中。
针对上面这个例子，当监听者使用`setMeasurements()`方法时就会调用`CurrentConditionsDisplay`
中的`update()`方法，从而做到"监听并触发事件"的效果。

但是上面的例子并不是一个很好的监听者事件的写法，因为通过遍历collections集合来
调用所有对象的`update()`方法所传入的参数是相同的，然而不同的对象所需要的参数是不同的，
这就导致了参数的冗余。

其改进的方式就是在`update()`方法中传入Observable对象，通过该对象获取事件对象
实际需要的参数。（可以通过java内置的观察者模式Observable类和Observer接口来实现）

### Summary

1. 监听类通过一个共同的接口来更新观察者。
2. 监听者和观察者之间用松耦合方式结合，监听者不知道观察者的细节，只知道观察者实现了了观察者接口。
