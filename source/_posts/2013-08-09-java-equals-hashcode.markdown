---
layout: post
title: "在java中复写equals和hashCode"
date: 2013-08-09 22:50
comments: true
categories: 
---
首先我们必须得知道什么是hash code。总的来说，Java中的集合（Collection）有两类，一类是List，再有一类是Set。你知道它们的区别吗？前者集合内的元素是有序的，元素可以重复；后者元素无序，但元素不可重复。那么这里就有一个比较严重的问题了：要想保证元素不重复，可两个元素是否重复应该依据什么来判断呢？这就是Object.equals方法了。但是，如果每增加一个元素就检查一次，那么当元素很多时，后添加到集合中的元素比较的次数就非常多了。也就是说，如果集合中现在已经有1000个元素，那么第1001个元素加入集合时，它就要调用1000次equals方法。这显然会大大降低效率。于是，Java采用了哈希表的原理。哈希（Hash）实际上是个人名，由于他提出一哈希算法的概念，所以就以他的名字命名了。哈希算法也称为散列算法，是将数据依特定算法直接指定到一个地址上。如果详细讲解哈希算法，那需要更多的文章篇幅，我在这里就不介绍了。初学者可以这样理解，hashCode方法实际上返回的就是对象存储的物理地址（实际可能并不是）。这样一来，当集合要添加新的元素时，先调用这个元素的hashCode方法，就一下子能定位到它应该放置的物理位置上。如果这个位置上没有元素，它就可以直接存储在这个位置上，不用再进行任何比较了；如果这个位置上已经有元素了，就调用它的equals方法与新元素进行比较，相同的话就不存了，不相同就散列其它的地址。所以这里存在一个冲突解决的问题。这样一来实际调用equals方法的次数就大大降低了，几乎只需要一两次。所以，Java对于eqauls方法和hashCode方法是这样规定的：1、如果两个对象相同，那么它们的hashCode值一定要相同；2、如果两个对象的hashCode相同，它们并不一定相同。
<!-- more -->
重上面的描述能看出，hash code一般是在例如set map等这样的集合中用到，因为它们通常需要通过hash code去判断两个对象是否相等。而如果不是在集合中用，其实复写不复写hash code意义都不大，不过作为一种良好的编程习惯，最好还是两个同时复写。下面我们来聚几个例子：
``` java
class Person {
    String name;
    int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Person person = (Person) o;

        if (age != person.age) return false;
        if (name != null ? !name.equals(person.name) : person.name != null) return false;

        return true;
    }

    @Override
    public String toString() {
        return "name : " + name + "; age : " + age;
    }
}
```
Person类中没有复写hashCode
``` java
public class test {
    public static void main(String[] args) {
        Person a = new Person("Adam", 27);
        Person b = new Person("Adam", 27);

        System.out.println(a.equals(b));  // true

        Set<Person> persons = new HashSet<Person>();
        persons.add(a);
        persons.add(b);
        System.out.println(persons.contains(a));  // true
        System.out.println(persons.contains(b));  // true
        System.out.println("size : " + persons.size());  // 2
    }
}
```
所以它们虽然对象里面的值相等，但是并不被认为是同一个对象，所以可以加到set中去。如果我们加上了hashCode：
``` java
class Person {
    String name;
    int age;

    Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int hashCode() {
        int result = name != null ? name.hashCode() : 0;
        result = 31 * result + age;
        return result;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;

        Person person = (Person) o;

        if (age != person.age) return false;
        if (name != null ? !name.equals(person.name) : person.name != null) return false;

        return true;
    }

    @Override
    public String toString() {
        return "name : " + name + "; age : " + age;
    }
}
```
那么下面这段代码的执行结果就应该是：
``` java
public class test {
    public static void main(String[] args) {
        Person a = new Person("Adam", 27);
        Person b = new Person("Adam", 27);

        System.out.println(a.equals(b));  // true

        Set<Person> persons = new HashSet<Person>();
        persons.add(a);
        persons.add(b);
        System.out.println(persons.contains(a));  // true
        System.out.println(persons.contains(b));  // true
        System.out.println("size : " + persons.size());  // 1
    }
}
```
1、equals()相等的两个对象，hashcode()一定相等； 

2、equals()不相等的两个对象，却并不能证明他们的hashcode()不相等。换句话说，equals()方法不相等的两个对象，hashcode()有可能相等。（我的理解是由于哈希码在生成的时候产生冲突造成的）。 

3、hashcode()不等，一定能推出equals()也不等；

4、hashcode()相等，equals()可能相等，也可能不等。
其实上面代码中的equals和hashCode函数都是Intellij自动生成的，以后编写都可以按照这种方法来。