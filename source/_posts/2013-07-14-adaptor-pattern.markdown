---
layout: post
title: "深入浅出设计模式之适配器模式(Adaptor Pattern)与外观模式(Facade Pattern)"
date: 2013-07-14 10:31
comments: true
categories: 
---
一、适配器模式
我们每个人在生活中都经历过这样的事，手中的电子产品和插座要求的接口类型不相符，比如插座要求的是三孔，而自己的插头是两孔，这个时候就需要一个电源适配器插在原先的插座上，然后再适配器的另一端暴露一个两孔的插孔供我们使用。对于客户来说，不管原来插座式几孔的，他只在乎自己能用的是两孔的就行了，其它的都交给了适配器去做。
<!-- more -->

{% img /images/adaptor_pattern-1.png %}

1、首先是美国标准的电源接口和实现
``` java
public interface TwoHolePlugIn {
    public void provideElectricityForTwoHole();
}
```
``` java
public class USAStandardTwoHolePlugIn implements TwoHolePlugIn {
    @Override
    public void provideElectricityForTwoHole() {
        System.out.println("USA Standard Electricity");
    }
}
```

2、其次是中国标准的接口和实现
``` java
public interface ThreeHolePlugIn {
    public void provideElectricityForThreeHole();
}
```
``` java
public class ChinaStandardThreeHolePlugIn implements ThreeHolePlugIn {
    @Override
    public void provideElectricityForThreeHole() {
        System.out.println("China Standard Electricity");
    }
}
```
3、我们想把美国的接口通过适配器转换为中国接口，就需要一个适配器类，它提供中国接口，但实际供电的是美国插座
``` java
public class TwoHolePlugInAdaptor implements TwoHolePlugIn {
    private TwoHolePlugIn plugInAdaptor;

    public TwoHolePlugInAdaptor(TwoHolePlugIn plugInAdaptor) {
        this.plugInAdaptor = plugInAdaptor;
    }

    @Override
    public void provideElectricityForTwoHole() {
        plugInAdaptor.provideElectricityForTwoHole();
    }
}
```
4、这时可以开始充电了
``` java
public class StarCharging {
    public static void main(String[] args) {
        TwoHolePlugIn twoHole = new USAStandardTwoHolePlugIn();
        twoHole.provideElectricityForTwoHole();

        ThreeHolePlugIn threeHole = new ChinaStandardThreeHolePlugIn();
        threeHole.provideElectricityForThreeHole();

        TwoHolePlugIn adaptor = new TwoHolePlugInAdaptor(twoHole);
        adaptor.provideElectricityForTwoHole();
    }
}
```
运行结果:
```
USA Standard Electricity
China Standard Electricity
USA Standard Electricity
```
从上面我们可以看到，客户只关心两孔接口，剩余的交给了适配器去做，而适配器不仅可以适配三孔插座，另一个适配器可以适配五孔插座，唯一需要统一的就是这两个适配器暴露给客户的接口类型一样就可以了。这样做的好处是解耦了客户和被适配者的具体实现，封装了架构中变化的部分。

二、外观模式
顾名思义，给客户提供一个更好地外观(接口)。例如有一个家庭影院，如果你想看电影，需要首先打开电视，打开蓝光DVD，调暗灯光这三步。这时有一个家庭影院类HomeMovie:
``` java
public class HomeMovie {
	public void turnOnTV() {
		...
    }

    public void turnOnBlueRayDVD() {
    	...
    }

    public void turnDownLight() {
    	...
    }
}
```
如果每次看电影都需要做这么多事情，而且它们都是固定不变的，是多么麻烦的事啊。这时我们就可以利用外观模式来解决这个问题，方法是提供另外一个接口，把这三步一次做完，这样客户只需做一件事情(比如按一次按钮)，一切都搞定了。新类如下：
``` java
public class HomeMovie {
	public void turnOnTV() {
		...
    }

    public void turnOnBlueRayDVD() {
    	...
    }

    public void turnDownLight() {
    	...
    }

    public void watchMovie() {
    	turnOnTV();
    	turnOnBlueRayDVD();
    	turnDownLight();
    }
}
```
好处是简化了客户调用，提供了一个干净的接口。值得注意的是外观模式没有封装如何类，只是简化了接口。同时也隔离了变化与不变化的地方，例如我换了一台电视机，但是客户只关心watchMovie成功与否，所以客户端代码无需任何改变。

迪米特法则(最少知识原则):
1、一个软件实体要尽可能的只与和它最近的实体进行通讯。通常被表述为：talk only to your immediate friends ( 只和离你最近的朋友进行交互)

“talk”，其实就是对象间方法的调用。这条规则表明了对象间方法调用的原则：
(1)  调用对象本身的方法；
(2)  调用通过参数传入的对象的方法；
(3)  在方法中创建的对象的方法；
(4)  所包含对象的方法。

上面的4点看起来有点别扭，下面通过一个具体的例子，就可以对上述4条guideline有进一步感性的认识：
``` java
 1 public class Car {
 2   Engine engine;
 3   
 4   public Car() {
 5     //initialize engine,etc.
 6   }
 7 
 8   public void start(Key key) {
 9     Doors doors = new Doors();
10     boolean authorized = key.turns();
11   
12     if(authorized) {
13       engine.start();
14       updateDashboardDisplay();
15       doors.lock();
16      }
17 
18     public void updateDashboardDisplay() {
19       //update display
20     }
21 }
```
下面对start()方法中的语句进行分析：
第10行－key.turns()：符合上述的第（2）条，key对象是通过参数传入start()方法的。
第13行－engine.start()：符合上述的第（4）条，engine对象是包含在Car的对象之中的。
第14行－UpdateDashboardDisplay()：符合上述的第（1）条，UpdateDashboardDisplay()方法是Car对像自身的方法。
第15行－doors.lock()：符合上述的第（3）条，doors对象是在start()方法中创建的对象。

接下来看一个违反Principle of Least Knowledge的例子：
``` java
1 public float getTemp() {
2   Thermometer thermometer = station.getThermometer();
3   return thermometer.getTemperature();
4 }
```
上面的方法中station对象是immediate friends。但是上面的代码却从station对象中返回了一个Thermometer对象，然后调用了thermometer对象的getTemperature()方法，违反了Principle of Least Knowledge。

下面对上面的方法作出符合Principle of Least Knowledge的改进：
```java
1 public float getTemp() {
2   return station. getTemperature();
3 }
```
我们在Station类中添加一个方法getTemperature()。这个方法将调用Station类中含有的Thermometer对象的getTemperature()。这样getTemp()方法就只知道Station对象而不知道Thermometer对象。

总结：笛米特法则告诉我们要尽量只和离自己最近的对象进行交互。离自己最近的对象包括：自身包含的对象，方法中创建的对象，通过参数传进的对象，还有自己本身。