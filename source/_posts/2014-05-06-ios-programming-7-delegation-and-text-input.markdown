---
layout: post
title: "iOS programming 7 (Delegation and Text Input)"
date: 2014-05-06 21:01
comments: true
categories: 
---
##UIResponder##
这一章主要讲解了iOS中的delegation.为了使这个概念更加清楚,我们通过一个例子来讲解.在上一章代码的基础上,我们在UITabBarController的第一个viewcontroller上加一个UITextField,这个textfield允许用户输入. 在创建完UITextField之后,我们把它加到viewcontroller的view中去:

<!-- more -->

```objective-c
- (void)loadView {
    BNRHypnosisterView *backgroundView = [[BNRHypnosisterView alloc] init];
    
    CGRect rect = CGRectMake(40, 70, 240, 30);
    UITextField *textField = [[UITextField alloc] initWithFrame:rect];
    
    [backgroundView addSubview:textField];
    
    self.view = backgroundView;
}
```
运行程序,可以看到屏幕中间有一个输入框:

 {% img /images/ios-7_1.png 300 600%}
 
当点击输入框的时候,键盘会自动跳出来,为了理解这个动作,我们首先需要理解一个叫做`first  responder`的原理.

在UIKit framework中有一个抽象类`UIResponder`,他是`UIView`,`UIViewController`,`UIApplication`的共同父类.在这个抽象类中定义了许多事件,比如touch event, motion events.由继承这个抽象类的子类选择去实现.在UIWindow中,有一个叫做`firstResponder`的指针,被它指向的对象用来处理这些事件.

{% img /images/ios-7_2.png 300 600%}

当用户点击UITextField的时候,这个`firstResponder`就指向了UITextField,代表这个`UITextField`变成了一个first responder,这时键盘就会弹出来,当`firstResponder`不在指向它的时候键盘就退回去.

##Delegation && Protocol##
在Delegate pattern中主要有object和delegate,它们的关系如下:

{% img /images/ios-7_3.png 300 600%}

object会有一个指针指向delegate,在object需要做一些事之前或者做完一些事之后会发送消息给delegate,然后delegate再完成一些其他的操作.例如,UITextField在用户输入完成后需要隐藏键盘,这时UITextField会给viewcontroller发送消息,案后这个viewcontroller可以完成隐藏键盘的操作.为了让WUSTLHypnosisViewController成为UITextField的delegate,我们需要加上下面这条语句:

```objective-c
textField.delegate = self;
```

因为这句话是在WUSTLHypnosisViewController里面,所以self指的是viewcontroller.这里有一个疑问,我们应该基于什么样的消息机制建立object和delegate之间的通信呢,这里就需要引出另外一个概念 -- protocol. Protocol定义了object能发送给delegate的消息,delegate负责实现protocol中的一些方法, 如果一个类实现了Protocol中的一些方法,我们称之为遵守协议.下面是一个Protocol的示例:

```objective-c
@protocol UITextFieldDelegate <NSObject>

@optional
- (BOOL) textFieldShouldBeginEditing:(UITextField *)textField;
- (BOOL) textFieldDidBeginEditing:(UITextField *)textField;
- (BOOL) textFieldShouldEndEditing:(UITextField *)textField;
- (BOOL) textFieldShouldClear:(UITextField *)textField;
```

上面的代码定义了一个Protocol,它其实类似于Java中的interface,都是只声明方法签名,留给子类去实现.它们分别在不同的时候被自动触发,例如`textFieldShouldBeginEditing`会在用户选中输入框的时候触发.上面<NSObject>的意思是遵守NSObject的协议,即这个Protocol中包含了所有NSObject Protocol中定义的方法.@optional代表下面的方法是可选实现的,非强制.当object尝试给delegate发送消息之前,它会发送另一个消息`respondToSelector`确认delegate是否实现了相应的方法.

下面我们来实现一个Protocol的方法,在这个方法里面我们调用另一个方法把用户输入随机打印20遍在屏幕上,然后清空输入框并且隐藏键盘.

```objective-c
- (BOOL)textFieldShouldReturn:(UITextField *) textField {
    [self drawHypnoticMessage:textField.text];
    textField.text = @"";
    [textField resignFirstResponder];
    return YES;
}
```
这个方法在用户输入完成点击键盘上Return时候被触发.倒数第二句话是用来通过设置UITextField不为firstResponder来达到隐藏键盘的目的.

下面是随机打印的代码:

```objective-c
- (void) drawHypnoticMessage:(NSString *) message {
    for (int i = 0; i < 20; i++) {
        UILabel *messageLabel = [[UILabel alloc] init];
        messageLabel.backgroundColor = [UIColor clearColor];
        messageLabel.textColor = [UIColor redColor];
        messageLabel.text = message;
        
        [messageLabel sizeToFit];
        
        int width = (int) (self.view.bounds.size.width - messageLabel.bounds.size.width);
        int x = arc4random() % width;
        
        int height = (int) (self.view.bounds.size.height - messageLabel.bounds.size.height);
        int y = arc4random() % height;
        
        CGRect frame = messageLabel.frame;
        frame.origin = CGPointMake(x, y);
        messageLabel.frame = frame;
        
        [self.view addSubview:messageLabel];
        
        UIInterpolatingMotionEffect *motionEffect = [[UIInterpolatingMotionEffect alloc] initWithKeyPath:@"center.x" type:UIInterpolatingMotionEffectTypeTiltAlongHorizontalAxis];
        motionEffect.minimumRelativeValue = @(-25);
        motionEffect.maximumRelativeValue = @(25);
        [messageLabel addMotionEffect:motionEffect];
        
        motionEffect = [[UIInterpolatingMotionEffect alloc] initWithKeyPath:@"center.y" type:
                        UIInterpolatingMotionEffectTypeTiltAlongHorizontalAxis];
        motionEffect.minimumRelativeValue = @(-25);
        motionEffect.maximumRelativeValue = @(25);
        [messageLabel addMotionEffect:motionEffect];
    }
```