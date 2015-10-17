---
layout: post
title: "一般递归与尾递归(Tail Recursion)"
date: 2013-07-09 21:58
comments: true
categories: 
---

最近在读一篇文章的时候无意间接触到了尾递归，感觉他比一般的递归好用，有点意思，顾将之记录下来。

首先需要了解的是到底什么是尾递归？
这个问题其实很简单，顾名思义，尾递归就是在一个方法的尾部、且在函数返回前的最后一步调用函数本身。为了更好地理解，还是上代码吧(以著名的斐波拉契为例)：
<!-- more -->
一般的递归：
``` java
private static int fibonacci(int number) {
    if (number == 0 || number == 1) {
        return number;
    }
    return number * fibonacci(number - 1);
}
```
在return返回前做了两件事，他们的顺序是：1、 计算fibonacci(number - 1)的值  2、 将fibonacci(number - 1)的值与number相乘。只有当第二步完成之后才返回。所以这里fibonacci方法最后做的事其实是一步乘法操作。

尾递归：
``` java
private static int fibonacci(int number, int result) {
    if (number == 0) {
        return 0;
    }
    if (number == 1) {
        return result;
    }
    return fibonacci(number - 1, result * number);
}
``` 
这里的最后一步还是在执行递归操作，所以这个方法就是一个尾递归。

那么尾递归的好处是什么呢？其实在了解这个之前我们不妨换一种思路，看看一般递归的坏处是什么!我们都知道，递归之所以如此臭名昭著，不仅在于它的不好理解，更在于它对内存的大量消耗。因为递归的执行其实是在不断地将一次执行未完成的参数、变量和返回地址等储存在栈中，而由于每个线程在执行代码时，都会分配一定尺寸的栈空间(Windows系统中为1M),这些信息再少也会占用一定空间，成千上万个此类空间累积起来，自然就超过线程的栈空间了，于是就会抛出StackOverFlowException。
尾递归的递归过程：
``` java
Rescuvie(5) 
{5 * Rescuvie(4)} 
{5 * {4 * Rescuvie(3)}} 
{5 * {4 * {3 * Rescuvie(2)}}} 
{5 * {4 * {3 * {2 * Rescuvie(1)}}}} 
{5 * {4 * {3 * {2 * 1}}}} 
{5 * {4 * {3 * 2}}} 
{5 * {4 * 6}} 
{5 * 24} 
120 
```

但是尾递归不会有上面的烦恼，为什么呢？因为尾递归在每次执行下一次递归之前，本次计算的结果已经算出来了，通过递归调用把本次得到的值传给下一次计算。所以，本次递归中所有的参数、变量、返回地址等都变得没有意义。所以编译器不会为尾递归分配多个栈空间，而是只分配一个，后面的操作都在这个栈空间里面进行。
``` java
TailRescuvie(5) 
TailRescuvie(5, 1) 
TailRescuvie(4, 5) 
TailRescuvie(3, 20) 
TailRescuvie(2, 60) 
TailRescuvie(1, 120) 
120 
```
那么接下来你也许会问，我们需要做什么去优化尾递归？答案是，我们无需做任何事，因为对尾递归的优化都是由语言本身决定的，很不幸的是，JVM并不支持对尾递归的优化，所以简而言之，在Java中即使使用尾递归，程序还是会为每一次的递归调用分配栈空间，这样就和一般递归没有差别了。

最后贴上全部代码：
1、一般递归
``` java
package recursion_and_iteration;

public class Recursion_fibonacci {
    public static void main(String[] args) {
        System.out.println(fibonacci(6));
    }

    private static int fibonacci(int number) {
        if (number == 0 || number == 1) {
            return number;
        }
        return number * fibonacci(number - 1);
    }
}
```
2、尾递归
``` java
package recursion_and_iteration;

public class TailRecursion_fibonacci {
    public static void main(String[] args) {
        System.out.println(fibonacci(6, 1));
    }

    private static int fibonacci(int number, int result) {
        if (number == 0) {
            return 0;
        }
        if (number == 1) {
            return result;
        }
        return fibonacci(number - 1, result * number);
    }
}
```