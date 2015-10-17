---
layout: post
title: "深入浅出设计模式之状态模式(State Pattern)"
date: 2013-07-16 22:39
comments: true
categories: 
---
在现实生活中，常常遇到许多与状态相关的问题，一些状态因为一些行为的触发转移到下一个状态。就拿一个糖果机为例子吧，糖果机有四种状态：没有25分钱、有25分钱、售出糖果和糖果售罄。由不同的触发条件到达另一种状态.例如如果没有25分钱，投入25分钱，会达到有25分钱的状态，其他状态如下图所示：

{% img /images/state_pattern-1.png %}

<!-- more -->

如果我们为这个状态转移图建模，也许会写出像下面这样的代码
``` java
public class GumballMachine {
    final static int SOLD_OUT = 0;
    final static int NO_QUARTER = 1;
    final static int HAS_QUARTER = 0;
    final static int SOLD = 3;
    int state = SOLD_OUT;
    int count = 0;

    public GumballMachine(int count) {
        this.count = count;
        if (count > 0) {
            state = NO_QUARTER;
        }
    }

    public void insertQuarter() {
        if (state == HAS_QUARTER) {
            System.out.println("You can't insert another quarter");
        } else if (state == NO_QUARTER) {
            state = HAS_QUARTER;
            System.out.println("You inserted a quarter");
        } else if (state == SOLD_OUT) {
            System.out.println("You can't insert a quarter, the machine is sold out");
        } else if (state == SOLD) {
            System.out.println("Please wait, we are already giving you a gumball");
        }
    }

    public void ejectQuarter() {
        if (state == HAS_QUARTER) {
            System.out.println("Quarter returned");
            state = NO_QUARTER;
        } else if (state == NO_QUARTER) {
            System.out.println("You haven't inserted a quarter");
        } else if (state == SOLD_OUT) {
            System.out.println("You can't eject, you haven't inserted a quarter yet");
        } else if (state == SOLD) {
            System.out.println("Sorry, you have turned the crank");
        }
    }

    public void turnCrank() {
        if (state == HAS_QUARTER) {
            System.out.println("You turned...");
            state = SOLD;
            dispense();
        } else if (state == NO_QUARTER) {
            System.out.println("You turned but there's not quarter");
        } else if (state == SOLD_OUT) {
            System.out.println("You turned, but there's no gumballs");
        } else if (state == SOLD) {
            System.out.println("Turn twice does not give you another gumball");
        }
    }

    private void dispense() {
        if (state == HAS_QUARTER) {
            System.out.println("No gumball dispensed");
        } else if (state == NO_QUARTER) {
            System.out.println("You need to pay first");
        } else if (state == SOLD_OUT) {
            System.out.println("No gumball dispensed");
        } else if (state == SOLD) {
            System.out.println("A gumball comes rolling out of the slot");
            count--;
            if (count == 0) {
                System.out.println("Oops, out of gumballs");
                state = SOLD_OUT;
            } else {
                state = NO_QUARTER;
            }
        }
    }
}
```
它在每一个行为发生之前先判断当前的状态，再根据此状态做出下一步判断。但这种设计违反了许多OO设计原则。首先是没有封装变化，即状态。其次是违反了开闭原则，因为只要进来一个新的状态，这里面所有方法都得加上一条if语句。最后这种设计不符合面向对象。

上面的做法是通过行为和当前状态决定下一步状态是什么，这样，状态就成了行为决定下一个状态的条件。我们不妨换个方式，以状态为中心，让行为作为条件决定下一个状态。这样就能把行为作为状态的行为，同时把行为都封装在了状态类中，一个类的变化不会影响其他行为，如果有新的行为和状态加进来，也可以把他们封装起来，对于调用者来说并没有差别。

通过使用状态模式，上面的糖果机代码会像下面这样：
1、首先定义所有状态的接口。如果有共同操作，就在父类实现，同时定义接口为抽象类。如果没有，就定义纯接口。
``` java
public interface State {
    public void insertQuarter();
    public void ejectQuarter();
    public void turnCrank();
    public void dispense();
}
```

2、根据四种不同的状态定义四个不同的状态类
``` java
public class HasQuarterState implements State {
    Random randomWinner = new Random(System.currentTimeMillis());
    GumballMachine gumballMachine;

    public HasQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }

    @Override
    public void insertQuarter() {
        System.out.println("You can't insert another quarter");
    }

    @Override
    public void ejectQuarter() {
        System.out.println("Quarter returned");
        gumballMachine.setState(gumballMachine.getNoQuarterState());
    }

    @Override
    public void turnCrank() {
        System.out.println("You turned...");
        int winner = randomWinner.nextInt(10);
        if (winner == 0 && gumballMachine.getCount() > 1) {
            gumballMachine.setState(gumballMachine.getWinnerState());
        } else {
            gumballMachine.setState(gumballMachine.getSoldState());
        }
    }

    @Override
    public void dispense() {
        System.out.println("No gumball dispensed");
    }
}
```
``` java
public class NoQuarterState implements State {
    private GumballMachine gumballMachine;

    public NoQuarterState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }

    @Override
    public void insertQuarter() {
        System.out.println("You inserted a quarter");
        gumballMachine.setState(gumballMachine.getHasQuarterState());
    }

    @Override
    public void ejectQuarter() {
        System.out.println("You haven't inserted a quarter");
    }

    @Override
    public void turnCrank() {
        System.out.println("You turned but there's not quarter");
    }

    @Override
    public void dispense() {
        System.out.println("You need to pay first");
    }
}
```
``` java
public class SoldOutState implements State {
    GumballMachine gumballMachine;
    public SoldOutState(GumballMachine gumballMachine, int count) {
        this.gumballMachine = gumballMachine;
        refill(count);
    }

    private void refill(int count) {
        gumballMachine.setCount(count);
    }

    @Override
    public void insertQuarter() {
        System.out.println("You can't insert a quarter, the machine is sold out");
    }

    @Override
    public void ejectQuarter() {
        System.out.println("You can't eject, you haven't inserted a quarter yet");
    }

    @Override
    public void turnCrank() {
        System.out.println("You turned, but there's no gumballs");
    }

    @Override
    public void dispense() {
        System.out.println("No gumball dispensed");
    }
}

```
``` java
public class SoldState implements State {
    GumballMachine gumballMachine;

    public SoldState(GumballMachine gumballMachine) {
        this.gumballMachine = gumballMachine;
    }

    @Override
    public void insertQuarter() {
        System.out.println("Please wait, we are already giving you a gumball");
    }

    @Override
    public void ejectQuarter() {
        System.out.println("You can't eject, you haven't inserted a quarter yet");
    }

    @Override
    public void turnCrank() {
        System.out.println("You turned, but there's no gumballs");
    }

    @Override
    public void dispense() {
        gumballMachine.releaseBall();
        if (gumballMachine.getCount() > 0) {
            gumballMachine.setState(gumballMachine.getNoQuarterState());
        } else {
            System.out.println("Oops, out of gumballs");
            gumballMachine.setState(gumballMachine.getSoldOutState());
        }
    }
}
```

3、使用到状态的类
``` java
public class GumballMachine {
    State soldOutState;
    State noQuarterState;
    State hasQuarterState;
    State soldState;

    State state = soldOutState;
    int count = 0;

    public GumballMachine(int numberGumballs) {
        soldOutState = new SoldOutState(this, 5);
        noQuarterState = new NoQuarterState(this);
        hasQuarterState = new HasQuarterState(this);
        soldState = new SoldState(this);

        this.count = numberGumballs;
        if (count < 0) {
            state = noQuarterState;
        }
    }
    public void insertQuarter() {
        state.insertQuarter();
    }

    public void ejectQuarter() {
        state.ejectQuarter();
    }

    public void turnCrank() {
        state.turnCrank();
    }

    public void dispense() {
        state.turnCrank();
        state.dispense();
    }

    public void releaseBall() {
        System.out.println("A gumball comes rolling out the slot..,");

        if (count != 0) {
            count--;
        }
    }

    public State getHasQuarterState() {
        return hasQuarterState;
    }

    public State getSoldOutState() {
        return soldOutState;
    }

    public State getNoQuarterState() {
        return noQuarterState;
    }

    public State getSoldState() {
        return soldState;
    }

    public void setState(State state) {
        this.state = state;
    }

    public int getCount() {
        return count;
    }

    public void setCount(int count) {
        this.count = count;
        state = noQuarterState;
    }
}
```
state变量存放了当前的状态，对于这个类来说只关心其他状态是否实现了State接口。通过直接调用这些状态类中共有的方法来达到状态的转移，相当于把自己不转移状态，而是委托给状态类自己去做。它的结构图如下：

{% img /images/state_pattern-2.png %}

这里的Context相当于GumballMachine类。值得注意的是，如果需要创建许多Context实例，而又要共享之间的状态，就需要把状态变量设为静态。

一句话总结状态模式:封装基于状态的行为，并将行为委托到当前状态。

