---
layout: post
title: "Java中Comparable和Comparator实现对象比较"
date: 2013-07-27 20:20
comments: true
categories: 
---
当需要排序的集合或数组不是单纯的数字型时，通常可以使用Comparator或Comparable，以简单的方式实现对象排序或自定义排序。下面通过两个例子分别用Comparable和Comparator实现对User对象中年龄排序。
 
<!-- more -->
 
1.通过实现Comparable接口，根据User的年龄进行排序
``` java
public class ComparableUser implements Comparable {  
  
    private String id;  
    private int age;  
  
    public ComparableUser(String id, int age) {  
        this.id = id;  
        this.age = age;  
    }  
  
    public int getAge() {  
        return age;  
    }  
  
    public void setAge(int age) {  
        this.age = age;  
    }  
  
    public String getId() {  
        return id;  
    }  
  
    public void setId(String id) {  
        this.id = id;  
    }  
  
    public int compareTo(Object o) {  
        return this.age - ((ComparableUser) o).getAge();  
    }  
  
    /** 
     * 测试方法 
     */  
    public static void main(String[] args) {  
        ComparableUser[] users = new ComparableUser[] {  
                new ComparableUser("u1001", 25),  
                new ComparableUser("u1002", 20),  
                new ComparableUser("u1003", 21) };  
        Arrays.sort(users);  
        for (int i = 0; i < users.length; i++) {  
            ComparableUser user = users[i];  
            System.out.println(user.getId() + " " + user.getAge());  
        }  
    }  
  
} 
```

2. 通过实现Comparator接口，根据User的年龄进行排序
``` java
public class User {  
  
    private String id;  
    private int age;  
  
    public User(String id, int age) {  
        this.id = id;  
        this.age = age;  
    }  
  
    public int getAge() {  
        return age;  
    }  
  
    public void setAge(int age) {  
        this.age = age;  
    }  
  
    public String getId() {  
        return id;  
    }  
  
    public void setId(String id) {  
        this.id = id;  
    }  
} 
```
``` java
public class UserComparator implements Comparator {  
  
    public int compare(Object arg0, Object arg1) {  
        return ((User) arg0).getAge() - ((User) arg1).getAge();  
    }  
  
    /** 
     * 测试方法 
     */  
    public static void main(String[] args) {  
        User[] users = new User[] { new User("u1001", 25),  
                new User("u1002", 20), new User("u1003", 21) };  
        Arrays.sort(users, new UserComparator());  
        for (int i = 0; i < users.length; i++) {  
            User user = users[i];  
            System.out.println(user.getId() + " " + user.getAge());  
        }  
    }  
}
```
选择Comparable接口还是Comparator？
 一个类实现了Comparable接口则表明这个类的对象之间是可以相互比较的，这个类对象组成的集合就可以直接使用sort方法排序。
Comparator可以看成一种算法的实现，将算法和数据分离，Comparator也可以在下面两种环境下使用：
1、类的设计师没有考虑到比较问题而没有实现Comparable，可以通过Comparator来实现排序而不必改变对象本身
2、可以使用多种排序标准，比如升序、降序等。