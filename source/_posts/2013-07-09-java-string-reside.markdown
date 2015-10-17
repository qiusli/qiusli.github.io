---
layout: post
title: "Java中的字符串驻留"
date: 2013-07-09 19:11
comments: true
categories: 
---

最近在工作的时候，一句再正常不过的代码String a = “hello” + “world”;被改成了new StringBuilder().append(“hello”).append(“world”)；当时就比较疑惑这样做的好处，后来到网上查找了一番之后才清楚这与Java中的字符串驻留机制有关，那么什么是驻留呢？
<!-- more -->
顾名思义，驻留就是在内存中保留（在Java中，我们通常称驻留对象的地方为驻留池，不过它也是内存的一部分），它不仅存在于Java中，在C#中同样存在。那么我就写几个例子来讲解什么叫Java中字符串的驻留：
``` java
public class test {
    public static void main(String[] args) {
        String a = "abc";
        String b = "abc";
        System.out.println(a == b);  // true
        String c = new String("abc");
        System.out.println(a == c);  // false
        System.out.println(a.equals(c)); // true
    }
}
```

上面这段代码在执行完String a = “abc”这一句的时候会在内存中创建一个值为abc的String类型对象。当执行下一句代码，即String b = “abc”的时候，java会首先去驻留池里面查找是否有值为abc的字符串对象，如果有就让b引用执行那个对象，如果没有就新创建一个并且将其存放在驻留池中。所以，不难理解，当程序执行到第三句话的时候会返回true，我们知道==在java中比较的是对象的引用指向的对象的内存首地址是否一样，而a和b指向的是同一个对象，所以会返回true。继续往下走，当程序执行到String c = new String(“abc”)这句话的时候，java做的事包括： 检查abc这个字符串对象是否在驻留池中，如果存在就把它当做值，然后再在堆上创建一个String类型的对象放到堆中（我们都知道在java中对象是放在堆中，对象的引用是存放在栈中）。所以这句话其实可能创建了2个对象(如果abc已经在驻留池中了，就只是在堆中创建了一个对象)。同时通过new String()创建出来的字符串对象是不会被放到驻留池中的。你也许会想，有没有一种方法让我把在堆中创建的对象放到驻留池中去呢？答案是有的！java提供了一个方法叫做intern()，如果执行c.intern()，会首先把c指向的对象放到驻留池中，然后返回指向这个对象的引用。那么，以下代码会输出什么呢？
``` java
package recursion_and_iteration;

public class test {
    public static void main(String[] args) {
        String a = "abc";
        String b = new String("abc");
        System.out.println(a == b.intern());  // true      
        System.out.println(a == b);  // false  
    }
}
```

当然是true！不过它的执行过程还是值得说一下的，重点在b.intern();这句话上。经过上面的讲解你也许会想过程应该是首先到堆中创建一个值为abc的String对象，然后将这个对象放到驻留池中。那么如果驻留池中已经存在值为abc的字符串对象了呢？那么b.intern会直接返回驻留池中的对象，所以这里会返回true。继续向下执行，System.out.println(a == b);会返回false，因为在执行b.intern();这句话的时候，实际上是直接返回了驻留池中的对象，所以对原本b指向的堆中的对象没有影响，所以a == b会返回false。

我通过上面这个例子简单讲解了java中的字符串驻留，那么现在回到文章开始部分的疑惑去，为什么使用StringBuilder而不是简单地使用”+”来连接字符串呢？经过上面的讲解，你可能会猜测StringBuilder用了字符串驻留，而”+”不是。恭喜你，你答对了，加10分。但是你也许并不知道使用”+”的时候tricky的地方在哪里。继续往下看。

原因在于使用+连接字符串每次都生成新的对象，而且是在堆内存上进行，而堆内存速度比较慢(相对而言)，那么再大量连接字符串时直接+是不可取的，当然需要一种效率高的方法。Java提供的StringBuffer和StringBuilder就是解决这个问题的。区别是前者是线程安全的而后者是非线程安全的。所以促使我写这篇博客的问题的原因就找到了。此外，值得注意的一点是，驻留池是不会被GC回收的，它会在程序运行期间一直保留。

最后我还想再说点题外话，请看下面这段程序：
``` java
public class test {
    public static void main(String[] args) {
        String a = "a";
        String b = "b";
        String c = "ab";
        String d = "a" + "b";
        String e = a + "b";
        String f = a + b;
        System.out.println(c == d);  // true      
        System.out.println(c == e);  // false     
        System.out.println(c == f);  // false  
        System.out.println(d == e);  // false     
        System.out.println(d == f);  // false    
        System.out.println(e == f);  // false  
    }
}
```
c == d输出true，因为c和d都是字符串常量，他们的值在编译时就确定了。而所有涉及到引用的地方都是在运行时才确定值的，所有下面会全部输出为false。