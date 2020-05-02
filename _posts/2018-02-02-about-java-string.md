---
layout: post
title: "Thinking in Java String"
date: 2018-02-02 06:02:14
image: 'http://res.cloudinary.com/degzyaemb/image/upload/c_scale,w_750/v1517372364/superman2.png'
description: 《Thinking in java》关于Java字符串处理的总结。
category: 'Java'
tags:
- Java
- String
twitter_text:
introduction: 《Thinking in java》关于Java字符串处理的总结。
---

昨天在看Java面试题的时候看到了一道关于String的题目。
```java
class StringEqualTest {

    public static void main(String[] args) {
        String s1 = "Programming";
        String s2 = new String("Programming");
        String s3 = "Program";
        String s4 = "ming";
        String s5 = "Program" + "ming";
        String s6 = s3 + s4;
        System.out.println(s1 == s2);
        System.out.println(s1 == s5);
        System.out.println(s1 == s6);
        System.out.println(s1 == s6.intern());
        System.out.println(s2 == s2.intern());
    }
}
```


## About String

> String对象是不可变得。查看JDK文档你就会发现，String类中每一个看起来会修改String值的方法，
实际上都是创建了一个全新的String对象，以包含修改后的字符串内容。而最初的String对象则丝毫未定。 ---from 《Thinking In Java》

我们都知道，在使用初始化一个String类型的对象的时候，引用会被放入栈中，而其字符串则会被置入常量池中。
如下的表达式中, 引用 `s` 被置入栈中，"Hello" 则进入常量池，`new String("Hello")` 这个对象则被放入堆中。

`String s = new String("Hello");`

然而对 `s` 进行 `+ "World"` 的操作时，实际上是在常量池中重新生成了一个 "Hello World" 字段，而"Hello"保持原样。

记得之前在网上看到一道实际面试题。

**问题**：`String s = new String("Test")`初始化的时候，创建了几个对象？

**答案** ：大部分人会认为只创建一个对象，而实际上，一个是字符串字面量"Test"所对应的、驻留（intern）在一个全局共享的字符串常量池中的实例，
另一个是通过`new String(String)`创建并初始化的、内容与"Test"相同的实例(堆中)。

## Difference between String、StringBuilder and StringBuffer

String的不可变性带来了一定的效率问题。为String对象重载 "+" 操作符就是一个例子。
当使用String来进行 "+" 操作的时候，通过 `javap -c ...` 生成的JVM字节码中可以看出，实际上，
编译器自动引入了`java.lang.StringBuilder`类，并且调用`StringBuilder`中的`append()`方法。

听起来好像String就是简化的StringBuilder，但是其实当你循环使用String的 "+" 操作时，
每一次的循环编译器都会重新生成一个新的StringBuilder，进而占据内容空间，减慢执行速度。

简单来说，String拥有只读特性，而StringBuilder创建的字符串可以进行修改。StringBuilder丰富全面的方法，
包括`insert()、replace()、substring()、append()和reverse()`等，StringBuffer与StringBuilder非常相似，
但是StringBuffer对所有的方法进行的`synchronized`，即所谓的线程安全，因而开销会大一点。
相比之下使用StringBuilder操作更加的快速。

