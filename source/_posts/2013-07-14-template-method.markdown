---
layout: post
title: "深入浅出设计模式之模板方法模式(Template Method)"
date: 2013-07-14 14:41
comments: true
categories: 
---
模板方法模式，顾名思义，必定有一个类提供了一个模板，完成一件工作。其他类使用这个模板完成自己的工作，但是由于各个类之间所要完成的工作之间并不完全一致，所有在模板方法类中会可以留下一些抽象方法供其他类自己来实现，这样通过模板方法达到的效果就产生了区别。例如，如果我要泡一杯咖啡，需要的步骤是
<!-- more -->

1、把开水煮沸；
2、用沸水冲泡咖啡；
3、把咖啡倒进杯子；
4、向咖啡中加牛奶和糖

那么如果我同时想泡一杯茶，步骤也许是

1、把开水煮沸；
2、用沸水侵泡茶叶；
3、把茶倒进杯子；
4、加柠檬

其实我们可以观察到，这上面的许多步骤都是相同或相似的，那么这时，我们就可以弄一个模板方法出来了。

1、首先是模板类
``` java
public abstract class Beverage {
    public void prepareBeverage() {
        boilWater();
        brew();
        pourInCup();
        addCondiment();
    }

    private void pourInCup() {
        System.out.println("Pouring in cup");
    }

    private void boilWater() {
        System.out.println("Boiling water");
    }

    protected abstract void brew();

    protected abstract void addCondiment();
}
```

模板类里面定义了准备的四个步骤，其中倒饮料进杯子和烧水都是相同的，所以这两步在父类中实现了。剩余的酿造和加作料留作子类自己实现。

2、茶类，继承自模板方法类
``` java
public class Tea extends Beverage {
    @Override
    protected void brew() {
        System.out.println("Stepping the tea");
    }

    @Override
    protected void addCondiment() {
        System.out.println("Adding lemon");
    }
}
```

咖啡类，继承自模板方法类

``` java
public class Coffee extends Beverage {
    @Override
    protected void brew() {
        System.out.println("Dripping coffee through filter");
    }

    @Override
    protected void addCondiment() {
        System.out.println("Adding sugar and milk");
    }
}
```

它们实现了父类中的抽象方法。

3、开始制造
``` java
public class Start {
    public static void main(String[] args) {
        Beverage tea = new Tea();
        tea.prepareBeverage();
        System.out.println();
        Beverage coffee = new Coffee();
        coffee.prepareBeverage();
    }
}
```

运行结果:
```
Boiling water
Stepping the tea
Pouring in cup
Adding lemon

Boiling water
Dripping coffee through filter
Pouring in cup
Adding sugar and milk
```

其实，模板方法是定义了一个算法步骤，并允许子类为一个或多个步骤提供实现。这样算法之存在于一个地方，容易修改，隔离了变化(子类)与不变化(父类)的地方。其实在实际编程中，我们也用到了很多模板方法，例如我们常常会为很多对象排序，这中间会用到compareTo方法，需要自己去实现它。当时现完后就会用Java提供的Collections.sort()方法来排序。是因为sort中用到了compareTo方法提供的实现来完成目的，而Colletions中的sort方法其实是一个模板方法。

接下来要介绍的东西叫做“钩子(hook)”。它是定义在父类中的一个默认方法，子类可以视情况将它覆盖。它为模板方法中可选的实现提供了一个接口，也就是说，模板方法类中已经实现了的方法是每个子类必须的，模板方法类中的抽象方法是必须由子类实现的，钩子就是默认的选择，子类可以不修改它，或者覆盖它。例如：
1、我在父类中留下了一个hook，即customerRequiresCondiment方法，如果其他类不管它，默认返回true，对结果没有影响。但是对于blacktea的人来说需要询问一下
``` java
public abstract class Beverage {
    public void prepareBeverage() {
        boilWater();
        brew();
        pourInCup();
        if (customerRequiresCondiment()) {
            addCondiment();
        }
    }

    private void pourInCup() {
        System.out.println("Pouring in cup");
    }

    private void boilWater() {
        System.out.println("Boiling water");
    }

    protected boolean customerRequiresCondiment() {
        return true;
    }

    protected abstract void brew();

    protected abstract void addCondiment();
}
```

2、红茶，需要询问
``` java
public class BlackTea extends Beverage {

    @Override
    protected void brew() {
        System.out.println("Brew black tea");
    }

    @Override
    protected void addCondiment() {
        System.out.println("Add sugar");
    }

    @Override
    public boolean customerRequiresCondiment() {
        String flag = new Scanner(System.in).nextLine();
        if (flag.equalsIgnoreCase("y")) {
            return true;
        } else {
            return false;
        }
    }
}
```
3、开始制作
``` java
public class Start {
    public static void main(String[] args) {
        Beverage tea = new Tea();
        tea.prepareBeverage();

        System.out.println();

        Beverage coffee = new Coffee();
        coffee.prepareBeverage();

        Beverage blackTea = new BlackTea();
        blackTea.prepareBeverage();
    }
}
```
运行结果：
``` text
Boiling water
Stepping the tea
Pouring in cup
Adding lemon

Boiling water
Dripping coffee through filter
Pouring in cup
Adding sugar and milk

Boiling water
Brew black tea
Pouring in cup
y
Add sugar
```
上面倒数第二行那个y是在控制台输入的。