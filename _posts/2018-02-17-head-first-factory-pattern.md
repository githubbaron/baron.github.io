---
layout: post
title: "Head First Factory Pattern"
date: 2018-02-17 07:36:52
image: 'http://res.cloudinary.com/degzyaemb/image/upload/c_scale,w_750/v1518871929/The_social_network4_vuj2um.png'
description: 《Head First Design Pattern》中关于工厂模式的介绍。
category: 'design pattern'
tags:
- design pattern
- factory pattern
twitter_text:
introduction: 《Head First Design Pattern》中关于工厂模式的介绍。
---

工厂模式有三种分类，其中包括*简单工厂模式*、*工厂方法模式*以及*抽象工厂模式*。
### 1. 简单工厂模式
> 又称为静态工厂方法(Static Factory Method)模式，
简单工厂模式专门定义一个类来负责创建其他类的实例，代码中可以根据参数的不同返回不同类的实例，
并且被创建的实例通常都具有共同的父类。但是*简单工厂模式*其实不是一个设计模式，反而比较像是一种编程习惯。

这里将使用
<a href="http://design-patterns.readthedocs.io/zh_CN/latest/creational_patterns/simple_factory.html">
简单工厂模式</a> 的模式分析。

1. 简单工厂模式最大的问题在于工厂类的职责相对过重，增加新的产品需要修改工厂类的判断逻辑，这一点与开闭原则是相违背的。
2. 简单工厂模式的要点在于：当你需要什么，只需要传入一个正确的参数，就可以获取你所需要的对象，而无须知道其创建细节。

*个人认为只有懂得该模式的应用场景，明白其存在的意义，才能更好的理解和践行。*
#### 应用场景：
1. JDK类库中广泛使用了简单工厂模式，如工具类java.text.DateFormat，它用于格式化一个本地日期或者时间。<br>
`public final static DateFormat getDateInstance(int style,Locale locale);`
2. Java加密技术 (获取不同加密算法的密钥生成器)<br>
`KeyGenerator keyGen=KeyGenerator.getInstance("DESede");`

对于《Head First Design Pattern》一书来说，简单工厂模式就是当需要不同类型的披萨时，
传入PizzaFactory不同的参数（"cheese"、"clam"、"veggies"）等来获取。
但是这种方式在加入新的类型的披萨的时候，就需要改动工厂类的判断逻辑。

```java
class PizzaFactory {
    public Pizza createPizza(String type) {
        if (type.equals("cheese"))
            return new CheesePizza();
        else if (type.equals("clam"))
            return new ClamPizza();
        return null;
    }
}

abstract class Pizza {
    protected String name;
    public abstract void eat();
}

class ClamPizza extends Pizza {
    public ClamPizza() { name = "clam pizza"; }
    public void eat() { System.out.println("Eating " + name); }
}

class CheesePizza extends Pizza {
    public CheesePizza() { name = "cheese pizza"; }
    public void eat() { System.out.println("Eating " + name); }
}

public class Client {
    public static void main(String []args) {
        PizzaFactory factory = new PizzaFactory();
        Pizza pizza = factory.createPizza("clam");
        pizza.eat();
    }
}

```

上述代码你可以看出当你需要添加veggies或者pepperoni类型的披萨时，你需要改变PizzaFactory中的代码。
这一点将在*工厂方法模式*中进行修改。

### 2. 工厂方法模式

> 工厂方法模式(Factory Method Pattern)又称为工厂模式，
也叫虚拟构造器(Virtual Constructor)模式或者多态工厂(Polymorphic Factory)模式，
它属于类创建型模式。在工厂方法模式中，工厂父类负责定义创建产品对象的公共接口，
而工厂子类则负责生成具体的产品对象，这样做的目的是将产品类的实例化操作延迟到工厂子类中完成，
即通过工厂子类来确定究竟应该实例化哪一个具体产品类。

通过上述披萨的例子来理解就是，由于简单工厂模式的变动会导致修改代码中的逻辑判断，
通过工厂模式进行修改，就是将`PizzaFactory`设置为接口并定义其必须实现的方法，创建`CheesePizzaFactory`、
`ClamPizzaFactory`等实现该接口，必要时就通过该子工厂类来创建不同类型的pizza。
使用该方法来进行改进后，当你需要添加新类型pizza的时候，你只需要添加相应的产品和工厂即可，
而不需要去改动工厂中的逻辑代码。

#### 应用场景

1. JDBC中的工厂方法

```java
class Template{
    public void connect() {
        Connection conn=DriverManager.getConnection("jdbc:microsoft:sqlserver://localhost:1433; DatabaseName=DB;user=sa;password=");
        Statement statement=conn.createStatement();
        ResultSet rs=statement.executeQuery("select * from UserInfo");
    }
}
```

下面通过改写pizza的例子来更好的理解：
```java
interface PizzaFactory {
    Pizza createPizza();
}

class CheesePizzaFactory implements PizzaFactory {
    public Pizza createPizza() { return new CheesePizza(); }
}

abstract class Pizza {
    protected String name;
    public abstract void eat();
}

class CheesePizza extends Pizza {
    public CheesePizza() { name = "cheese pizza"; }
    public void eat() { System.out.println("Eating " + name); }
}

public class Client {
    public static void main(String []args) {
        PizzaFactory factory = new CheesePizzaFactory();
        Pizza pizza = factory.createPizza();
        pizza.eat();
    }
}

```

但是工厂模式会有相应的缺点，即**会使系统中类的个数将成对增加，在一定程度上增加了系统的复杂度，
有更多的类需要编译和运行，会给系统带来一些额外的开销。**

### 3. 抽象工厂模式

> 抽象工厂模式(Abstract Factory Pattern)：提供一个创建一系列相关或相互依赖对象的接口，而无须指定它们具体的类。抽象工厂模式又称为Kit模式，属于对象创建型模式。

抽象工厂模式的定义非常抽象，但是通过pizza的例子可以很好的理解其中的意思。

由于不同地区的披萨都有不同的配料，比如像chicago的奶酪披萨和new york的奶酪披萨就一定的区别。
这时候抽象工厂模式就排上用场了。将配料也设计成工厂的形式，并且不同的地区有不同的工厂类。
将需要的配料工厂注入进披萨工厂中，使其通过"组合"来完成（工厂方法模式是使用"继承"）。

```java
interface PizzaFactory {
    Pizza createPizza(PizzaIngredient ingredient);
}

class CheesePizzaFactory implements PizzaFactory {
    public Pizza createPizza(PizzaIngredient ingredient) { return new CheesePizza(ingredient); }
}

abstract class Pizza {
    protected String name;
    public abstract void eat();
}

class CheesePizza extends Pizza {
    private PizzaIngredient ingredient;
    public CheesePizza(PizzaIngredient ingredient) {
        name = "cheese pizza";
        this.ingredient = ingredient;
    }
    public void eat() {
        String result =  name + " with ";
        for (String ingre: ingredient.createIngredient())
            result += ingre + ", ";
        System.out.println("Eating " + result);
    }
}

interface PizzaIngredient { // 将pizza配料设计成工厂接口的形式
    ArrayList<String> createIngredient();
}

class NYPizzaIngredient implements PizzaIngredient{ // 不同的地区有不同的配料
    public ArrayList<String> createIngredient() {
        ArrayList<String> ingredient = new ArrayList<>();
        ingredient.add("double cheese");
        ingredient.add("milk");
        return ingredient;
    }
}

public class Client {
    public static void main(String []args) {
        PizzaFactory factory = new CheesePizzaFactory();
        // 将配料工厂注入进做pizza的工厂中
        Pizza pizza = factory.createPizza(new NYPizzaIngredient());
        pizza.eat();
    }
}

```

