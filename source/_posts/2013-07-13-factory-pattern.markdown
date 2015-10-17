---
layout: post
title: "深入浅出设计模式之工厂模式(Factory Method Pattern)"
date: 2013-07-13 14:01
comments: true
categories: 
---
工厂模式算是设计模式中用到的最多的一类了，对于它最直观的描述就是：父类定义一个创建对象的接口，由子类决定需要创建什么对象，这样让类把实例化推迟到了子类。下面来举个例子来阐述吧。

有一个披萨店，可以做各种各样的披萨，但位于不同地方的分店里的披萨味道会有些不同。如果利用工厂模式，就是：
<!-- more -->

1、有一个Pizza连锁店，它负责接收客户的订单(orderPizza)，之后就开始生产披萨
``` java
public abstract class PizzaStore {
    protected Pizza orderPizza(String type) {
        Pizza pizza = createPizza(type);

        pizza.prepare();
        pizza.bake();
        pizza.cut();
        pizza.box();

        return pizza;
    }

    protected abstract Pizza createPizza(String type);
}
```
上面留了一个createPizza方法，这就是一个工厂方法，它返回一个产品的基类类型，而具体的类型由子类自己决定。父类不管返回的是什么，只管对返回的产品做相应的操作。这样做的好处是隔离了变化与不变的地方，面向接口编程。

2、纽约的这家Pizza连锁店只生产纽约口味的披萨
``` java
public class NYPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String type) {
        if (type.equals("cheese")) {
            return new NYStyleCheesePizza();
        } else if (type.equals("veggie")) {
            return new NYStyleVeggiePizza();
        } else if (type.equals("clam")) {
            return new NYStyleClamPizza();
        } else {
            return null;
        }
    }
}
```

芝加哥的Pizza连锁店只生产芝加哥口味的披萨
``` java
public class ChicagoPizzaStore extends PizzaStore {
    @Override
    protected Pizza createPizza(String type) {
        if (type.equals("cheese")) {
            return new ChicagoStyleCheesePizza();
        } else if (type.equals("veggie")) {
            return new ChicagoStyleVeggiePizza();
        } else if (type.equals("clam")) {
            return new ChicagoStyleClamPizza();
        } else {
            return null;
        }
    }
}
```

3、同时有一个Pizza的抽象类，各种披萨都继承自他
``` java
public abstract class Pizza {
    protected String description;
    public void prepare() {
        System.out.println("Preparing " + description);
    }

    public void bake() {
        System.out.println("Baking " + description);
    }

    public void cut() {
        System.out.println("Cutting " + description);
    }

    public void box() {
        System.out.println("Boxing " + description);
    }
}
```
``` java
public class ChicagoStyleCheesePizza extends Pizza {
    public ChicagoStyleCheesePizza() {
        description = "Chicago Style Cheese Pizza";
    }
}
```
``` java
public class NYStyleCheesePizza extends Pizza {
    public NYStyleCheesePizza() {
        description = "New York Style Cheese Pizza";
    }
}
```
.
.
.
(这里就不一一列出了)

4、客户开始订披萨	
``` java
public class StartOrderingPizza {
    public static void main(String[] args) {
        PizzaStore chicagoPizzaStore = new ChicagoPizzaStore();
        chicagoPizzaStore.orderPizza("cheese");

        PizzaStore nyPizzaStore = new NYPizzaStore();
        nyPizzaStore.orderPizza("veggie");
    }
}
```
上面可以发现，在不同的地方的店里面点相同的披萨得到的是不同的。披萨店的类图如下：
{% img /images/factory_pattern-1.png %}

另外，出了工厂模式之外，还有一种叫做抽象工厂模式。此工厂模式生成各种不同类型的物品的产品家族。例如我们要生成各种各样的鸭子，这些鸭子来自同一个家族(实现了相同接口)，我们就可以用抽象工厂模式：
``` java
public abstract class AbstractDuckFactory {
    public abstract Quackable createMallardDuck();
    public abstract Quackable createRedheadDuck();
    public abstract Quackable createDuckCall();
    public abstract Quackable createRubberDuck();
}
```