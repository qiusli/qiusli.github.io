---
layout: post
title: "深入浅出设计模式之命令模式(Command Pattern)"
date: 2013-07-13 19:12
comments: true
categories: 
---
命令模式，顾名思义，一定是有大哥发命令，一定有小弟在执行这个命令，理解了这个，命令模式就不难了。命令模式里面分为三种角色，发送者、命令对象和接受执行者。比如说有一个遥控板，它可以控制室内的灯光，那么遥控板在这里就是一个发送者，它发送命令给命令对象，命令对象接到命令后就只管执行。下面来看这个命令模式的实现吧：
<!-- more -->
1、首先是命令对象，它是一个接口，由子类具体实现
``` java
public interface Command {
    public void execute();
    public void undo();
}
```

2、具体的命令，接收真实的接受者并运行接受者的方法来达到执行
``` java
public class LightOffCommand implements Command {
    private Light light;

    public LightOffCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.off();
    }

    @Override
    public void undo() {
        light.on();
    }
}
```
``` java
public class LightOnCommand implements Command {
    private Light light;

    public LightOnCommand(Light light) {
        this.light = light;
    }

    @Override
    public void execute() {
        light.on();
    }

    @Override
    public void undo() {
        light.off();
    }
}
```
他们都实现了Command，并且实现了execute方法和undo方法，并且封装了命令的接收者。这两个方法是命令模式里命令对象的标配，前者负责执行命令，后者负责撤销命令。其实它们都不直接完成命令，而是委托给通过构造函数传进来的真正的执行者去完成(这里是Light对象)。这个Command对象暴露给命令发送者就是2个接口，一个execute一个undo。命令发送者只管通过对命令对象的引用来执行这两个方法就是了，具体的都封装在命令对象中(例如LightOnCommand中)。
3、命令的真正接收和执行者
``` java
public class Light {
    public void on() {
        System.out.println("light's on");
    }

    public void off() {
        System.out.println("light's off");
    }
}
```
4、命令发送者
``` java
public class SimpleRemoteControl {
    private Command slot;

    public void setCommand(Command command) {
        this.slot = command;
    }

    public void buttonWasPressed() {
        slot.execute();
    }
}
```
发送命令
``` java
public class SimpleController {
    public static void main(String[] args) {
        SimpleRemoteControl simpleRemoteControl = new SimpleRemoteControl();
        simpleRemoteControl.setCommand(new LightOnCommand(new Light()));

        simpleRemoteControl.buttonWasPressed();
    }
}
```
创建一个命令发送者对象，然后设置一个要发送的命令，而这个命令里面又包含了具体的命令接受执行者。当发送者调用buttonWasPressed时，就调用命令对象的execute方法。相当于对命令对象发出了命令，具体的执行就看命令对象自己调用哪个接受者来执行了。所以对于命令发送者来说，根本不在乎所拥有的是什么命令对象，只要该对象实现了Command接口就行。具体是什么Command类型都是调用者决定的(例如这里的SimpleController)。

命令模式把“发出请求的对象”和“接受和执行请求的对象”分隔开来