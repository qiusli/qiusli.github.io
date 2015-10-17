---
layout: post
title: "深入浅出设计模式之装饰器模式(Decorator Pattern)"
date: 2013-07-12 14:53
comments: true
categories: 
---
今天该讲第三种设计模式 -- 装饰器模式。同样，从名字就可以看出这种模式必定有一个基础组件，用来被修饰，也会有至少一个装饰器来修饰基础组件。同时要求被装饰器修饰过的组件对外界来说还是一个组件，只不过在被修饰之后多出了一些以前没有的功能。装饰器模式可以看做下图：
<!--more -->
{% img /images/decorator_pattern-1.png %}

装饰器模式相对于一般的继承，做到了对类的行为的扩展的同时，避免了从父类继承来的不必要的东西，其实是使用了组合的方式来达到的。那么下面就来看看这种模式的具体实现吧。假如有一个咖啡店，卖和多种咖啡，而每种咖啡又可以加上不同的调料，例如奶昔、牛奶等。同时每种咖啡价格不一样，调料的价格也不一样，加过调料的咖啡的价格就是原本咖啡喝调料的总价。咋一看来，似乎继承是一种很好的方式：（如下图）
{% img /images/decorator_pattern-2.png %}

每种咖啡都实现了统一的接口，子类自己覆盖父类的description和cost方法，但是问题来了，这样的话就会产生很多的类，因为需要为每一种咖啡都建一个类，同时为每一个加了调料的咖啡也建一个类，这样就会产生类爆炸。正确的方式应该是使用装饰器模式，为每一种调料建一个类作为装饰器，这样就会有很多的装饰器类，如果有人想买一杯加了奶昔和豆浆的抹茶咖啡，就应该使用奶昔和豆浆来装饰这种咖啡，使用了组合提高了类的灵活性。下面来看看代码吧：
1、首先有一个饮料抽象类
``` java
public abstract class Beverage {
    String description = "unknown beverage";

    public String getDescription() {
        return description;
    }

    public abstract double cost();
}
```

2、三种不同的饮料
``` java
public class DarkRoast extends Beverage {
    public DarkRoast() {
        description = "Dark Roast";
    }

    @Override
    public double cost() {
        return .99;
    }
}
``` 
``` java
public class Expresso extends Beverage {
    public Expresso() {
        description = "Espresso";
    }

    @Override
    public double cost() {
        return 1.99;
    }
}
```
``` java
public class HouseBlend extends Beverage {
    public HouseBlend() {
        description = "House Blend Coffee";
    }

    @Override
    public double cost() {
        return .89;
    }
}
```

3、有一个装饰器的抽象类，它继承自饮料类
``` java
public abstract class CondimentDecorator extends Beverage {
    public abstract String getDescription();
}
``` 

4、三种不同的调料
``` java
public class Mocha extends CondimentDecorator {
    private final Beverage beverage;

    public Mocha(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + ", Mocha";
    }

    @Override
    public double cost() {
        return .20 + beverage.cost();
    }
}
```
``` java
public class Soy extends CondimentDecorator {
    private final Beverage beverage;

    public Soy(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + ", Soy";
    }

    @Override
    public double cost() {
        return .15 + beverage.cost();
    }
}
```
``` java
public class Whip extends CondimentDecorator {
    private final Beverage beverage;

    public Whip(Beverage beverage) {
        this.beverage = beverage;
    }

    @Override
    public String getDescription() {
        return beverage.getDescription() + ", Whip";
    }

    @Override
    public double cost() {
        return .10 + beverage.cost();
    }
}
```
我们可以看到每一种装饰器继承自CondimentDecorator，而CondimentDecorator继承自Beverage类，所以实际上每一种装饰器都是一种饮料。从每个装饰器内部都保留了一个饮料的引用就能看出，装饰器实际上实在‘接收’一种饮料，然后给他加点调料，然后返回加了调料的饮料。基础组件和装饰器的关系在咖啡这个例子里面可以通过下面这张图来描述：

{% img /images/decorator_pattern-3.png %}

4、点咖啡加调料的实际流程
``` java
public class StarBuzzCoffee {
    public static void main(String[] args) {
        Beverage beverage1 = new Expresso();
        System.out.println(beverage1.getDescription() + " &" + beverage1.cost());

        Beverage beverage2 = new DarkRoast();
        beverage2 = new Mocha(beverage2);
        beverage2 = new Mocha(beverage2);
        beverage2 = new Whip(beverage2);
        System.out.println(beverage2.getDescription() + " &" + beverage2.cost());

        Beverage beverage3 = new HouseBlend();
        beverage3 = new Soy(beverage3);
        beverage3 = new Mocha(beverage3);
        beverage3 = new Whip(beverage3);
        System.out.println(beverage3.getDescription() + " &" + beverage3.cost());
    }
}
```

运行结果：
``` text
Espresso &1.99
Dark Roast, Mocha, Mocha, Whip &1.49
House Blend Coffee, Soy, Mocha, Whip &1.34
```
当每次结果装饰器修饰过的基础组件在上面这段代码执行时，过程是这样的：

{% img /images/decorator_pattern-4.png %}