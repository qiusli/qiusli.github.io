---
layout: post
title: "Java中对象的浅拷贝与深拷贝"
date: 2013-07-09 19:17
comments: true
categories: 
---
Java中的拷贝方式分为深拷贝和浅拷贝。简单来说，深拷贝就是把一个对象中的所有值，如果被拷贝对象中有对其他对象的引用，那么这个引用指向的对象本身会被重新创建。浅拷贝和深拷贝类似，但是如果被拷贝对象中有对其他对象的引用，只是这个引用会被拷贝，而不是这个被引用的对象。

说起来有点绕口，那么我们就看看下面的图解吧：

{% img /images/deep_shallow_copy-1.png %}

shallow copy:

{% img /images/deep_shallow_copy-2.png %}

deep copy:

{% img /images/deep_shallow_copy-3.png %}

<!-- more -->
来看下面这段代码：
``` java
public class Car {
    private String brand;
    private int price;

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }
}
```
``` java
public class Person {
    private String name;
    private Car car;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Car getCar() {
        return car;
    }

    public void setCar(Car car) {
        this.car = car;
    }
}
```
``` java
public class test {
    public static void main(String[] args) {
        Car car1 = new Car();
        car1.setBrand("BMW");
        car1.setPrice(10000);
        Person person1 = new Person();
        person1.setCar(car1);
        person1.setName("person1");
        Person person2 = person1;
        person2.setName("person2");
        System.out.println(person1.getName()); // person2        
        System.out.println(person2.getName()); // person2       
        Car car2 = new Car();
        car2.setBrand("Benz");
        car2.setPrice(20000);
        person1.setCar(car2);
        System.out.println(person2.getCar().getBrand()); // Benz       
        System.out.println(person2.getCar().getPrice()); // 20000    
    }
}
```

重点在Person person2 = person1;这一句上，person1里面包括了一个对Car对象的引用，那么这句话是深拷贝还是浅拷贝呢？答案是什么都不是。它只是一个简单的引用传递，执行完这句话以后，person1和person2都指向了同一个person对象，所以无论谁去改变对象，另一个引用再去调用该对象的值都会发生改变。 好吧，言归正传，下面来实现一个浅拷贝。
``` java
public class Person implements Cloneable {
    private String name;
    private Car car;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Car getCar() {
        return car;
    }

    public void setCar(Car car) {
        this.car = car;
    }

    public Object clone() {
        Person person = null;
        try {
            person = (Person) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return person;
    }
}
```
``` java
public class CloneTest {
    public static void main(String[] args) {
        Car car1 = new Car();
        car1.setBrand("BMW");
        car1.setPrice(10000);
        Person originalPerson = new Person();
        originalPerson.setCar(car1);
        originalPerson.setName("originalPerson");
        Person clonePerson = (Person) originalPerson.clone();
        originalPerson.setName("originalPerson_1");
        originalPerson.getCar().setBrand("Benz");
        System.out.println(originalPerson.getName()); // originalPerson_1 
        System.out.println(originalPerson.getCar().getBrand()); // Benz 
        System.out.println(clonePerson.getName()); // originalPerson 
        System.out.println(clonePerson.getCar().getBrand()); // Benz 
    }
}
```
Car类不变，Person实现了Cloneable接口，然后重载了父类的clone方法，并且直接调用super.clone()方法来拷贝。但是值得注意的是，父类的clone只是浅拷贝，所以才会有上述的输出结果。那么，要想达到深拷贝，需要做些什么呢？ 其实答案已经很明显了，因为clone是浅拷贝，而Car中都是原始类型的变量，所以我们只需要让Car类也实现Cloneable接口，然后重载clone方法，然后回到Person类中，在clone的时候，加上car = car.clone()就行了。
``` java
package problems;

public class Car implements Cloneable {
    private String brand;
    private int price;

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public Object clone() {
        Car car = null;
        try {
            car = (Car) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return car;
    }
}
```
Person类中的clone:
``` java
public Object clone() {
        Person person = null;
        try {
            person = (Person) super.clone();
            car = (Car) car.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return person;
    }
``` 
然后我们回到test类中，再去看执行结果:
``` java
package problems;

public class CloneTest {
    public static void main(String[] args) {
        Car car1 = new Car();
        car1.setBrand("BMW");
        car1.setPrice(10000);
        Person originalPerson = new Person();
        originalPerson.setCar(car1);
        originalPerson.setName("originalPerson");
        Person clonePerson = (Person) originalPerson.clone();
        clonePerson.setName("clonePerson");
        clonePerson.getCar().setBrand("Benz");
        System.out.println(originalPerson.getName()); // originalPerson 
        System.out.println(originalPerson.getCar().getBrand()); // BMW 
        System.out.println(clonePerson.getName()); // clonePerson 
        System.out.println(clonePerson.getCar().getBrand()); // Benz 
    }
}
```
但是，在一些java原生类中，并没有实现clone方法，这表明他们是不能被拷贝的。这时，如果想要达到深度拷贝的目的，就需要这么做: car = new Car();然后把原来的car中的值全部一一重新set到新的car对象中。 其实，除了上面这种方法之外，还有一种方法能达到深拷贝的效果，那就是使用序列化。
``` java
package problems;

import java.io.Serializable;

public class Car implements Serializable {
    private String brand;
    private int price;

    public String getBrand() {
        return brand;
    }

    public void setBrand(String brand) {
        this.brand = brand;
    }

    public int getPrice() {
        return price;
    }

    public void setPrice(int price) {
        this.price = price;
    }

    public Object clone() {
        Car car = null;
        try {
            car = (Car) super.clone();
        } catch (CloneNotSupportedException e) {
            e.printStackTrace();
        }
        return car;
    }
}
```
``` java
package problems;

import java.io.*;

public class CloneTest {
    public static void main(String[] args) throws IOException, ClassNotFoundException {
        Car originalCar = new Car();
        originalCar.setBrand("BMW");
        originalCar.setPrice(10000);
        ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
        ObjectOutputStream objectOutputStream = new ObjectOutputStream(byteArrayOutputStream);
        objectOutputStream.writeObject(originalCar);
        objectOutputStream.flush();
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(byteArrayOutputStream.toByteArray());
        ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);
        Car cloneCar = (Car) objectInputStream.readObject();
        System.out.println(cloneCar.getBrand()); // BMW 
        System.out.println(cloneCar.getPrice()); // 10000 
        cloneCar.setBrand("Honda");
        cloneCar.setPrice(3000);
        System.out.println(originalCar.getBrand()); // BMW 
        System.out.println(originalCar.getPrice()); // 10000 
        System.out.println(cloneCar.getBrand()); // Honda 
        System.out.println(cloneCar.getPrice()); // 3000 
    }
}
``` 
在这里，我们首先让Car类实现Serializable接口使其能够序列化，其次就可以使用java中的io来传输对象了。序列化能够达到深拷贝目的的原因是，它首先将对象的整个对象树持久化，然后全部读出，每读出一次就得到一个全新的拷贝。但是这样做的坏处也不少，首当其冲的就是效率问题，序列化的方式比起clone方法的方式要慢不少。 最后需要说的是，我们究竟什么时候用浅拷贝，什么时候用深拷贝呢？答案就是如果一个对象中只包含原始类型的变量，那么就使用浅拷贝，如果类中有对其他类的引用，但是其他类是imutable的，仍然使用浅拷贝。如果有对其他类的引用，而其他类是可被修改的，这就不得不深拷贝了。