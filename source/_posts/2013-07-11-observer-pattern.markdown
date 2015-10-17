---
layout: post
title: "深入浅出设计模式之观察者模式(Observer Pattern)"
date: 2013-07-11 21:39
comments: true
categories: 
---
观察者模式，顾名思义，这个模式中必定有观察者和被观察者。打一个形象的比喻，一些在证券交易市场的股民眼巴巴地看着自己买的股票所代表的公司能放些风声出来，这样自己就可以更好地把握这个消息做出买或卖的决定。而上市公司为了加强广大股民的信心，鼓励大家增持股票也会不断地放出例如公司的新项目马上出来了，大家赶紧买啊这些消息。我们暂且不论消息的真假，但这确实一个非常贴切的例子。

<!-- more -->

观察者模式中的被观察者与观察者的对应关系是1:n，意思是一个被观察者对于n个观察者。被观察者叫做subject或observable，观察者叫做observer。被观察者可以通知观察者新的消息，但是观察者只是被动地等待。被观察者中维护着一系列的注册过的观察者，这些观察者可以通过添加进入被观察者的通知范围，也可以通过删除避免继续街道被观察者的通知。下面我们通过一个股票程序来阐述观察者和被观察者的结构：

1、被观察者的接口及其实现(红星集团会放出消息，期待股民增持自己的股票)
``` java
public interface CorperationSubject {
    public void registerObserver(InvesterObserver investerObserver);
    public void removeObserver(InvesterObserver investerObserver);
    public void notifyObservers();
}
```
``` java
import java.util.ArrayList;
import java.util.List;

public class HongtaSecurity implements CorperationSubject {
    private List<InvesterObserver> observerList;
    private int announcedIncome = 1000000;
    private int actualIncome = 500000;

        public HongtaSecurity() {
        this.observerList = new ArrayList<InvesterObserver>();
    }

    @Override
    public void registerObserver(InvesterObserver investerObserver) {
        observerList.add(investerObserver);
    }

    @Override
    public void removeObserver(InvesterObserver investerObserver) {
        int index = observerList.indexOf(investerObserver);
        if (index != -1) {
            observerList.remove(index);
        }
    }

    @Override
    public void notifyObservers() {
        for (InvesterObserver investerObserver : observerList) {
            investerObserver.update("明天股票会大涨！！");
        }
    }

    public int getAnnouncedIncome() {
        return announcedIncome;
    }

    public int getActualIncome() {
        return actualIncome;
    }
}
```


2、观察者的接口和实现(这里有两个实现，分别表示一个聪明的和一个笨的投资人。聪明的觉得红星是在耍小把戏，笨的投资者完全相信公司的话)
``` java
public interface InvesterObserver {
    void update(String s);
}
```
``` java
public class Investor_Smart implements InvesterObserver{
    private String message;

    @Override
    public void update(String message) {
        this.message = message;
        action();
    }

    private void action() {
        System.out.println("红星集团一定放的假消息在骗钱，明天一早就得抛");
    }
}
```
``` java
public class Investor_Dumb implements InvesterObserver {
    private String message;

    @Override
    public void update(String message) {
        this.message = message;
        action();
    }

    private void action() {
        System.out.println("明天一早就买进10000股红星证券");
    }
}
```
3、两种投资人在消息下来后做出的截然不同的反应
``` java
public class Anoncement {
    public static void main(String[] args) {
        HongtaSecurity hongtaSecurity = new HongtaSecurity();

        hongtaSecurity.registerObserver(new Investor_Dumb());
        hongtaSecurity.registerObserver(new Investor_Smart());

        hongtaSecurity.notifyObservers();
    }
}
```
运行输出：
``` text
明天一早就买进10000股红星证券
红星集团一定放的假消息在骗钱，明天一早就得抛
``` 
最后还有一种特别聪明的人，在集团放出话后，它真实地去查看了集团的公布的营业额和实际的营业额，发现实际的比公布的差了不少，判定公司一定是在做假账。
1、特别聪明的人
``` java
public class Investor_wickedlySmart implements InvesterObserver, Action {
    private String message;

    @Override
    public void update(String message) {
        this.message = message;
        action();
    }

    @Override
    public void action() {
        HongtaSecurity hongtaSecurity = new HongtaSecurity();
        if (hongtaSecurity.getActualIncome() <= hongtaSecurity.getAnnouncedIncome()) {
            System.out.println("红星在做假账，看来是假消息，明天一定得抛");
        } else {
            System.out.println("红星的话看来是真的，明天疯狂买进");
        }
    }
}
```	

在实际应用中，observer里面会包含对subject的应用，以便自己退出subject的通知或者再次加进去。观察者模式应用到了很多OO原则，例如通过隔离变化的观察者和不变化的被观察者来分离固定于不变，通过在被观察者中注册观察者来达到针对接口而非实现编程的目的，同时它应用了组合在被观察者中保持了对观察者的引用而没有用继承来实现。

其实，Java在util包中有Observable和Observer类，它们分别对应了上面的subject和observer的角色。所以可以说观察者模式在Java中是内置了的，由此可以看出这种模式的应用有多么宽广。