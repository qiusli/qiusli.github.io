---
layout: post
title: "深入浅出设计模式之单件模式(Singleton Pattern)"
date: 2013-07-13 15:21
comments: true
categories: 
---
单件模式是最简单的一种设计模式，因为从头到尾都只涉及到一个类。单件模式想要达到的目的是在程序运行的过程中只创建一个类的对象，在很多场合这都很有用，比如注册表程序，你不会想要很多对象来，因为那样会造成混乱且很占内存，一个对象足矣。那么应该怎么应用单件模式呢？例子很简单，代码如下：
<!-- more -->
``` java
public class Singleton {
	private static Singleton uniqueInstance;

	private Singleton(); //这里把构造函数的修饰符设为private是为了在禁止外部类创建Singleton类的对象，而只允许在Singleton类里面创建

	public static Singleton getInstance() {
		if (uniqueInstance == null) {
			uniqueInstance = new Singleton(); // 延迟初始化
		}

		return uniqueInstance;
	}
}
``` 
如果外部类想要用到Singleton类的实例，不能自己创建，而必须向Singleton类请求这个实例。值得注意的是，Singleton类中单间的实现用到了static，意味着不要去继承Singleton类，因为这样所有潜在的子类都能使用这个uniqueInstance，会造成很多混乱。
