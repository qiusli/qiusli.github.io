---
layout: post
title: "iOS Programming 6-2(UITabBarController)"
date: 2014-05-06 15:57
comments: true
categories: 
---
我们在上一章中讲到了如何使用viewcontroller,这里我讲讲解一种技术,它可以管理多个viewcontroller,并在这些viewcontroller之间切换,这项技术叫做UITabBarController.

在使用这项技术后,效果如下.我们可以看到在屏幕的下端有两个按钮可以互相切换,他们分别对应一个viewcontroller:

<!-- more -->

 {% img /images/6-2_1.png 300 600%}
 
我们来到AppDelegate,在里面创建一个UITabBarController,然后把另外两个viewcontroller加到这个UITabBarController里面去.最后把这个UITabBarController作为window的rootViewController,这样就完成了.

```objective-c
    WUSTLHypnosisViewController *hvc = [[WUSTLHypnosisViewController alloc] init];
    WUSTLReminderViewController *rvc = [[WUSTLReminderViewController alloc] init];
    
    UITabBarController *tabBarController = [[UITabBarController alloc] init];
    tabBarController.viewControllers = @[hvc, rvc];
    
    self.window.rootViewController = tabBarController;
```

我们发现,在每一个tab上都有一个图片,图片下面有一行介绍性的文字,这些都是可以改变和设置的.虽然UITabBarController管理着两个不同的viewcontroller,我们可以把viewcontroller连同tabbar上面的图片和文字想成一个整体,所以图片和文字就需要到相应的viewcontroller里面去设置,我们来到WUSTLHypnosisViewController类,在initWithNibName方法里面进行初始化:

```objective-c
        self.tabBarItem.title = @"Hypnosister";
        
        UIImage *image = [UIImage imageNamed:@"Hypno.png"];
        self.tabBarItem.image = image;
```
到另一个viewcontroller里面做同样的事,到这里为止就算完成了UITabBarController的设置了.