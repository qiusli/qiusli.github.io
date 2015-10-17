---
layout: post
title: "iOS Programming 2(objective-c)"
date: 2014-03-07 18:36
comments: true
categories: 
---
Objective-c中的方法调用其实是一种消息传送的机制，例如下面这段代码：

```objective-c
[partyInstance addAttendee:somePerson]
```

他表达的意思是发送addAttendee:消息给partyInstance对象，然后这个对象再去调用他的类中的addAttendee方法，somePerson在这里是参数。在运行时，消息被发送到对象，然后对象会向创建自己的那个类寻求帮助，并告诉这个类去运行消息中指定的方法(e.g. addAttendee)(这里需要注意的是，一切都是发生在运行时，而不是一些其他语言在编译时进行检查)。那么一个对象又是怎么知道自己属于哪个类呢？其实每个对象都有一个叫做'isa'的instance variable，当这个对象被创建的时候，这个isa就自动被初始化为指向创建他的那个类。这个instance variable叫做'isa'是有道理的，因为他是创建他的类的一个实例。他们的关系如下：

<!--more-->

{% img /images/objects.png 500 800%}

一个对象只响应在他的类中定义了的方法。因为这一切都是发生在运行时，所以xCode并不能在运行时(项目build的时候)发现到底这个对象能不能相应传进来的消息。有时xCode能判断出是否能响应，但是有时并不能，例如：

```objective-c
NSArray *items = @[@"A", @"B", @"C"];
id lastString = [items lastObject];
[lastString count];
```

上面首先创建一个array，里面装了3个string对象的引用，然后返回最后一个string引用给lastString，然后调用lastString的count方法，因为lastString其实是一个string，并没有count方法，所以xCode会抛出异常。在iOS开发中，异常被视为应该被程序员捕获并修改的错误，而不是被动地等待其在运行时抛出来，所以try...catch并不在iOS开发中流行。

在objective-c中，array并不存放对象本身，而是对象的引用，同时和其他语言的array不同的是，一个array中可以存放不同类型的对象引用。array中不能存放像int这样的primative值，如果要存放他，需要使用对应的包装对象(NSInteger)。其次，objective-c只支持单继承，每个类只能有一个父类。每当创建一个类的时候，两个文件会被创建.h和.m。.h是头文件，他定义类名，类的继承关系，实例变量和方法等，.m文件主要实现.h文件中定义的方法，在.m文件里需要import.h文件。

objective-c中有两种方式调用acccessor。当在.h文件中定义了instance variable和对应的getter方法之后，可以在.m文件中使用下面的方法调用：

```objective-c
[self ivarName];
```

或者直接使用"."调用：

```objective-c
self.ivarName;
```

以前我总以为这种调用是直接访问的实例变量，其实不然，这其实是在访问实例变量的getter方法，只不过方法的名字叫做ivarName罢了。

objective-c中有两个特殊的类型值得一提，instanceType和id。前者只能作为方法的返回类型使用，他匹配调用这个方法的对象的类型，通常在类的init方法中使用，例如：

```objective-c
- (instancetype)init {
    return [self initWithItemName:@"item"];
}
```

我们通常会使用下面这种方法来新建一个对象：

```objective-c
NSMutableArray *items = [[NSMutableArray alloc] init];
```

这样就不难发现，在最后调用init的时候，init方法返回了一个instanceType的类型，这个类型被自动匹配给了NSMutableArray。和instanceType不一样，id不仅可以作为方法的返回类型，而且还可以直接作为对象的声明类型，类似于Java中的Object，他指向任何对象：

```objective-c
id name = @"abc";
```