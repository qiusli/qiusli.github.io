---
layout: post
title: "如何在Java中创建Immutable的类"
date: 2013-08-08 21:30
comments: true
categories: 
---

首先，我们需要明白什么是Immutable的类。顾名思义，就是在对象创建后，它的状态不能改变。你首先也许会想到final这个关键字，因为它会使被修饰者要么不能被继承(修饰类)，要么不能被重新赋值(修饰字段)。下面我们来看一个简单的例子：
``` java
final class ImmutableClass_1 {
    private final int i = 0;
    private final double j = 1;

    int getI() {
        return i;
    }

    double getJ() {
        return j;
    }
}
```
<!-- more -->
这个类被final修饰，意味着它不能被继承，去除了子类修改状态的可能。字段i和j都被什么味final因为这它不能被重新赋值，所以它们都是不能改变的。最后这个类中没有任何的构造函数或者setter方法能对其中的字段做出改变。综上所述，这个类是一个不可变类。

下面让我们来看一个稍微复杂一些的例子:
``` java
final class MutableClass_1 {
    private final int i = 0;
    private final List<String> stringList = new ArrayList<String>();

    int getI() {
        return i;
    }

    List<String> getStringList() {
        return stringList;
    }
}
```
这个类和上面那个类很相似，类被final修饰，字段也被final修饰，但是它却并非immutable，问题出在stringList上。我们都知道在创建对象的时候，对象名只是一个指向真正对象的引用，我们可以通过这个引用去修改被指向的对象。这里的getStringList方法返回了这个对象的引用，于是这个对象就会有2个引用指向它，所以客户端可以通过下面这段代码来改变对象：
``` java
public class test {
    public static void main(String[] args) {
        MutableClass_1 mutableClass_1 = new MutableClass_1();
        System.out.println(mutableClass_1.getStringList());  // print '[]'

        mutableClass_1.getStringList().add("a new string");
        System.out.println(mutableClass_1.getStringList());  // print 'a new string'
    }
}
```
我们看到mutableClass_1这个对象中的字段被改变了，所以它不是immutable的。如果想要让上面这个MutableClass_1变成不可变的，就需要用到Java中的深拷贝和浅拷贝了。代码如下:
``` java
final class ImmutableClass_2 {
    private final int i = 0;
    private final List<String> original = new ArrayList<String>();

    int getI() {
        return i;
    }

    List<String> getStringList() {
        List<String> copy = new ArrayList<String>();
        for (String s : original) {
            copy.add(s);
        }
        return copy;
    }
}
```
我在返回这个list的时候使用了深拷贝，返回了一个与老的list值完全一样的一个新的list。这样，在客户端调用的时候，指向的是另一个list了。所以下面这段代码就不能修改这个类了。
``` java
public class test {
    public static void main(String[] args) {
        ImmutableClass_2 mutableClass_2 = new ImmutableClass_2();
        System.out.println(mutableClass_2.getStringList());   // print '[]'

        mutableClass_2.getStringList().add("a new string");
        System.out.println(mutableClass_2.getStringList());   // print '[]'
    }
}
```

其实在Java中有很多默认的类都是immutable的，例如String，Integer等。这就意味着，当你在使用String的一些方法的时候，其实它返回的是另一个对象，原对象并没有改变。比如：
``` java
public class test {
    public static void main(String[] args) {
        String a = "AbC";
        String b = a.toLowerCase();
        System.out.println(a);  // print 'AbC'
        System.out.println(b);  // print 'abc'
    }
}
```
总结起来，如果要创建一个‘不可变’类，需要遵守下面几条：

1、Class应该定义成final，避免被继承

2、所有的成员变量应该被定义成final

3、尽量不要暴露mutable的字段(例如list)，如果要暴露，使用深拷贝或浅拷贝

4、不要提供可以改变类状态(成员变量)的方法(get 方法不要把类里的成员变量让外部客服端引用,当需要访问成员变量时，返回成员变量的copy)

5、构造函数不要引用外部可变对象。如果需要引用外部可以变量，应该在构造函数里进行defensive copy