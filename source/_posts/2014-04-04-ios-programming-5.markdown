---
layout: post
title: "iOS Programming 5(Redrawing and UIScrollView)"
date: 2014-04-04 13:08
comments: true
categories: 
---
在上一篇博客中,我们创建了一个画同心圆的app,这篇博客将继续在那上面扩展,当用户在屏幕上点击的时候同心圆会改变颜色.此外,还将在app里面加上UIScrollView,让用户通过scroll的方式浏览比屏幕本身大的view.

##改变同心圆颜色##
既然是通过点击来改变同心圆的颜色,那么我们肯定会想到用touch event来实现,在用户touch之后需要改变颜色,我们可以通过创建一个公有变量,然后在touch方法里改变他的颜色. 我们现在项目的格局如下:

<!--more-->

 {% img /images/ios-5_1.png 300 600%}

首先我们在`BNRHypnosisterView.m`的头部创建一个property文件:

```objective-c
@interface BNRHypnosisterView ()

@property (strong, nonatomic) UIColor *circleColor;

@end

@implementation BNRHypnosisterView
...
@end
```
在这里创建property而不是在header文件里面创建的原因是隐藏这个变量.一个类的header文件是对所有类可见,一种默认的做法是如果变量是定义在header文件里面,那么就表示他可以和其他所有的类互动,如果我想要这个变量或者方法只在这个类中使用,那么可以把它定义在.m的category里面.这相当于对其他类隐藏了这个变量或方法,即使其子类都不能看到.

声明完之后就应该开始定义了,我们可以到`initWithFrame`方法里给它首先赋予一个默认值:

```objective-c

- (id)initWithFrame:(CGRect)frame
{
    self = [super initWithFrame:frame];
    if (self) {
        // Initialization code
        self.backgroundColor = [UIColor clearColor];
        self.circleColor = [UIColor lightGrayColor];
    }
    return self;
}

```

这样在程序启动的时候同心圆的颜色是灰色.接下来就应该实现touch even了, 当用户点击后就改变``circleColor``的颜色我们通过实现默认的``touchesBegan``方法来实现.

```objective-c

- (void) touchesBegan:(NSSet *)touches withEvent:(UIEvent *)event {
    float red = (arc4random() % 100) / 100.0;
    float green = (arc4random() % 100) / 100.0;
    float blue = (arc4random() % 100) / 100.0;
    
    UIColor *randomColor = [UIColor colorWithRed:red green:green blue:blue alpha:1];
    self.circleColor = randomColor;
}

```

上面这个方法就是在每次用户点击的时候都重新开始算rgb在``randomColor``中的分量,然后得到一个新的`randomColor`并把它赋值给circleColor.这样`circleColor`的颜色就变了,现在我们可以运行看看效果:

 {% img /images/ios-5_2.png 300 600%}
 
我们发现无论怎么点击屏幕,颜色都不会改变,这是为什么呢?这里有两个原因:

* UIView的drawRect方法默认情况下只在加载的时候被调用,所以无论我们怎么点击屏幕,这个方法都不会被再次调用,所以所有的view都没有被重新render.
* 在drawRect方法里面没有改变同心圆的颜色,改变的代码如下:

```objective-c

 // [[UIColor lightGrayColor] setStroke];
    [self.circleColor setStroke];

```

这里又涉及到iOS的一种机制:`run loop`. 它指的是当iOS app 在运行的时候,他会开始一个`run loop`,这个loop是专门用来监听事件的.当一次事件发生时(例如touch),`run loop`会找到相应的handler来处理这次事件,当这个事件被处理后,`run loop`继续执行. 在我们这个程序中,当`run loop`发现touch事件时,会停下来调用`touchesBegan`方法处理它,在处理完后重新回到`tun loop`中.在重新回到`run loop`后,它会检查那些在上次执行handler之后有改变并且需要重新render的view,然后`run loop`会发送drawRect:方法给这些view,然他们重新去render.但是为什么我们在点击屏幕后view并没有被重新render呢?原来这是因为需要被重新render的view不是系统默认的,如果他们是默认的(例如UIButton, UIText),render会自动执行,否则我们需要调用view的`setNeedsDisplay`方法来让他重新render.在`touchesBegan:`方法的最后一句,我们看到`circleColor`被重新赋值(其实是它的setter方法被调用了),那么我们可以重载它的setter方法,并在里面调用`setNeedsDisplay`方法.

```objective-c

- (void) setCircleColor:(UIColor *)circleColor {
    _circleColor = circleColor;
    [self setNeedsDisplay];
}

```

再次运行,一切OK.


 {% img /images/ios-5_3.png 250 500%}
 {% img /images/ios-5_4.png 250 500%}
 {% img /images/ios-5_5.png 250 500%}

##使用UIScrollView##
Scroll view通常是为比屏幕更大的view准备的,这样就能通过上下左右滚动来查看完整的图片.之前我们是直接把当前的view加到``UIWindow``中,不过为了实现scroll view,我们需要把当前view加到scroll view中,然后再把这个scroll view加到``UIWindow``中去.

 {% img /images/ios-5_UIScrollView.png 900 1200%}

然后到app delegate的``didFinishLaunchingWithOptions``里加上如下代码:

```objective-c

    CGRect screenRect = self.window.bounds;
    CGRect bigRect = screenRect;
    bigRect.size.width *= 2;
    bigRect.size.height *= 2;
    
    UIScrollView *scrollView = [[UIScrollView alloc] initWithFrame:screenRect];
    [self.window addSubview:scrollView];
    
    BNRHypnosisterView *hypnosisView = [[BNRHypnosisterView alloc] initWithFrame:bigRect];
    [scrollView addSubview:hypnosisView];

```

首先建立一个屏幕大小的CGRect作为``UIScrollView``的显示范围,然后建立一个两倍屏幕大小的CGRect作为view的显示范围,最后把view加到UIScrollView中.显示的效果如下:

 {% img /images/ios-5_6.png 300 600%}
 
最后,我们可以把它做的更好看一些,我们给`UIScrollView`加两个view,并让他们在不同的屏幕显示:


```objective-c

    CGRect screenRect = self.window.bounds;
    CGRect bigRect = screenRect;
    bigRect.size.width *= 2;

 UIScrollView *scrollView = [[UIScrollView alloc] initWithFrame:screenRect];
    [self.window addSubview:scrollView];
    
    BNRHypnosisterView *hypnosisView = [[BNRHypnosisterView alloc] initWithFrame:screenRect];
    scrollView.pagingEnabled = YES;
    [scrollView addSubview:hypnosisView];

    screenRect.origin.x += screenRect.size.width;
    BNRHypnosisterView *hypnosisView2 = [[BNRHypnosisterView alloc] initWithFrame:screenRect];
    [scrollView addSubview:hypnosisView2];
    
    scrollView.contentSize = bigRect.size;

```

其中`scrollView.pagingEnabled = YES`的意思是在左右滑动的时候有翻页的效果,屏幕不会停在两个view的中间.
 {% img /images/ios-5_7.png 300 600%}