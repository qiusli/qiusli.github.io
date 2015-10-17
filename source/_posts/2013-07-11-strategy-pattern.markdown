---
layout: post
title: "深入浅出设计模式之策略模式(Strategy Pattern)"
date: 2013-07-11 00:02
comments: true
categories: 
---

今天开始阅读Head First设计模式，老早就听说这是一本经典书籍，但是这类华丽的辞藻确实听过太多，每每在阅读的时候发现自己也许期望太高了或者自己的水平不足，并没有带给自己耳目一新的感觉。但是今天在阅读这本书的时候，终于找到了一种久违的兴奋感，而它更来自于对书中内容的由浅入深的理解和阅读的乐趣。例如，在第一章讲解策略模式的时候，在读到书的最后一页之前，我并不知道自己学习的是哪种模式，所以在学习的过程中
我能完全静下心来理解纯粹的内容，而不是一边试图去记住“策略模式”这四个字，一边去看“策略”应用在哪里，而在这个过程中往往偏离了书本想要表达的内容，及作者期望读者走的学习道路，致使书本的作用大大降低。从引导读者这一点上来说，Head First设计模式这本书做得非常好！
<!-- more -->

说完书的概况，下面就来说说书的第一章内容 -- 策略模式吧。策略模式这个名字听起来挺吓人的，策略二字似乎隐含着无尽的逻辑和‘勾心斗角’，但是take it easy，事实并没有这么复杂。简单来说，策略模式就是将一个对象的行为(通常被称为一个算法族)分离出来并将其封装，达到高复用的目的。

现在有一个真实的需求，一家玩具公司生产了一批鸭子，他们有很多种，比如一般的鸭子，诱饵鸭，火箭鸭等等(管它叫什么名字呢)。鸭子有‘叫’，‘飞’两种行为。对于设计者来说，最直观的设计方案是创造一个Duck的父类，里面有‘叫’和‘飞’两个方法，然后其他的鸭子就去继承。但是这样会造成一些子类继承了自己并不想要的东西，比如诱饵鸭继承了‘飞’，而它却不会飞。应用OO中‘分离变化与不变化的原则’，你也许又会想到将父类鸭子里面变化的行为分离出来，做成单独的接口，子类根据实际需要去实现。但问题又来了，如果一般鸭子和火箭鸭都会飞，那岂不是分别要实现同样的代码两次？那么应该怎么做呢？我们不妨在分离出来的接口下面再实现一层具体类，它们可以分别代表不同的类型，比如在‘叫’下面可以分为‘尖叫’，‘呱呱叫’和‘不叫’。‘飞’的行为与其类似。这样，所有鸭子的所有行为我们都知道了，剩下的就是应用组合，不同的鸭子根据需求找到对应的行为类组合起来。接下来是比较tricky的地方，会用到OO设计原则里的‘针对接口而不是针对实现编程’。既然是针对接口，那么这里只有两种接口 -- ‘叫’和‘飞’。虽然不同的行为被分离了出来，但是原来Duck父类还是保留了相应的行为，只不过这些行为变成了接口，意味着对所有子类提供了公共的接口，具体行为还得看子类自己的实现。同时，子类还能改变自身的行为，这样就大大地提高了灵活性。

下面是上面说的实现:
1、父类Duck
``` java
public abstract class Duck {
    protected QuackBehavior quackBehavior;
    protected FlyBehavior flyBehavior;

    protected void setQuackBehavior(QuackBehavior quackBehavior) {
        this.quackBehavior = quackBehavior;
    }

    protected void setFlyBehavior(FlyBehavior flyBehavior) {
        this.flyBehavior = flyBehavior;
    }

    protected void performQuack() {
        quackBehavior.quack();
    }

    protected void performFly() {
        flyBehavior.fly();
    }
}
```

2、‘叫’接口与‘飞’接口
``` java
public interface QuackBehavior {
    void quack();
}
```
``` java
public interface FlyBehavior {
    void fly();
}
```

3、具体的鸭子的行为的实现 -- 来自‘叫’和‘飞’接口
``` java
public class Quack implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("quack");
    }
}
```
``` java
public class Squeak implements QuackBehavior {
    @Override
    public void quack() {
        System.out.println("Squeak");
    }
}
```
``` java
public class FlyNoWay implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I can't fly");
    }
}
```
``` java
public class FlyWithWings implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I'm flying");
    }
}
```
``` java
public class FlyRocketPowered implements FlyBehavior {
    @Override
    public void fly() {
        System.out.println("I am flying with a rocket");
    }
}
```

4、子类实现父类Duck
``` java
public class ModelDuck extends Duck {
    public ModelDuck() {
        flyBehavior = new FlyNoWay();
        quackBehavior = new Quack();
    }
}
```
``` java
public class MallardDuck extends Duck {
    public MallardDuck() {
        quackBehavior = new Quack();
        flyBehavior = new FlyWithWings();
    }
}
```

5、应用
``` java
public class MiniDuckSimulator {
    public static void main(String[] args) {
        Duck duck = new MallardDuck();
        duck.performQuack();
        duck.performFly();

        Duck model = new ModelDuck();
        model.performQuack();
        model.performFly();

        model.setFlyBehavior(new FlyRocketPowered());
        model.performFly();
    }
}
``` 
这里我们可以看到model对象在被初始化后还能改变自身的行为，即model.setFlyBehavior(new FlyRocketPowered());同时如果系统扩展，更多的类被加了进来，我们需要做的仅仅是加上一些新的行为，同时不会对原有类造成影响，这就是这个模式的精妙之处。

这个时候我们可以回到第二部看看，它创建了所有鸭子的所有行为，然后我们根据一定的‘策略’选择特定的行为组合起来。也许这就是策略模式名称的由来吧。

最后是上面提到的类的依赖关系：
{% img /images/strategy_pattern.png %}
