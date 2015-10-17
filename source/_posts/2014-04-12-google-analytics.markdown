---
layout: post
title: "google analytics"
date: 2014-04-12 20:52
comments: true
categories: 
---
最近在iOS的final project中用到了google analytics来分析用户行为,自己也做了一些research,今天就把它记录在这里.

在介绍之前先说说这个iOS app的概况,他是一个时间记录仪,有显示当前时间(ClockViewController)和stopwatch的功能(StopWatchViewController),可以在app下面的tab切换.我今天要做的是首先为显示时间的view加一个track,然后再到stopwatch里面加两个track用户点击按钮的功能.

<!--more-->

 {% img /images/ga-10.png 300 600%}
 
##什么是Google Analytics##
Google Analytics是一款用来监测网页或者app中用户行为的软件,这里的用户行为包括用户在哪个页面停留了多久,哪些功能用户使用得最多,哪个版本的app最受用户欢迎等内容.


##如何使用Google Analytics##
###Step 1. 点击首页banner上面的Admin,进入管理页面###
 {% img /images/ga-1.png 600 1200%}
 
上图是Google Analytics的dashboard,里面有三个column,分别是account,property和view.

* account: 一个account可以是其他account的member,比如我的account可以是客户或者其他合作者的account的一个member.今天我不会深入讨论这个,所以可以暂时放一放.
* property: 一个property代表一个被track的网站或者app,所以如果你要对这个account新增一个网页或者app去track,你需要新建一个property.
* view: 如果你的app有多个版本,你可以创建多个view来分别track他们.

###Step 2. 新建一个property###
只需要点击property,然后会出来一个下拉框,然后点击里面的`create new property`

 {% img /images/ga-2.png 600 1200%}
 
之后你会被带到下面这个页面:

 {% img /images/ga-3.png 600 1200%}
 
因为今天我们使用它来track一个iOS app,所以这里选择`Mobile app`,然后填上`App Name`,点击`Get Tracking ID`,然后会被带到下面这个页面:

 {% img /images/ga-4.png 600 1200%}
 
在这个页面你会得到你app的专属tracking id,每个应用都有自己的一个tracking id,在你的app中会使用到这个id,之后我们会具体谈到其原理和使用.

现在下载Google Analytics iOS SDK,点击第一个下载连接,而不是`Download with admob features`.这样你就得到了一个SDK包,接下来就是引用包了.

###Step 3. 引用Google Analytics iOS SDK###
解压刚才下载下来的那个包:

 {% img /images/ga-5.png 600 1200%}
 
打开GoogleAnalytics下面的library文件夹:

 {% img /images/ga-6.png 600 1200%}
 
把下面几个文件拷贝到你的项目中去,注意在拷贝的时候要选择` Copy items into destination group’s folder and Add the files to the Clock target.`,这样才能保证文件被真正拷贝到了项目里面,而不是只产生了对文件的引用.

   * GAI.h
   * GAITracker.h
   * GAITrackedViewController.h
   * GAIDictionaryBuilder.h
   * GAIFields.h
   * GAILogger.h

{% img /images/ga-7.png 300 600%}

接下来把`libGoogleAnalyticsServices.a`文件拖到项目的framework文件夹下,其实你把它放到项目的哪儿都无所谓,只是这样更好管理一些.点击项目名称(Clock),然后在右边的窗口中点击`build phases`,展开`Link Binary With Libraries`点击左下角的`+`然后在搜索栏中输入CoreData,然后选中并点击`Add`.重复上面的过程,并把`SystemConfiguration.framework`和`libz.dylib`引用进来.

{% img /images/ga-8.png 600 1200%}

最后需要修改`supporting files`文件夹下面的`Clock-Prefix.pch`文件,把下面两个语句加入到文件中:

```objective-c
#import "GAI.h"
#import "GAIFields.h"
```

####Step 3.1 GAI (Google Analytics for iOS)####
GAI是一个top-class,它利用一个tracking id创建一个共有的tracker,然后这个tracker被暴露给所有类,这样一个项目中就只有一个tracker.

###Step 4. 初始化Tracker###
在AppDelegate类的`application:didFinishLaunchingWithOptions:`方法中插入以下代码:

```objective-c
    // 1
    [GAI sharedInstance].trackUncaughtExceptions = YES;
    
    // 2
    [[GAI sharedInstance].logger setLogLevel:kGAILogLevelVerbose];
    
    // 3
    [GAI sharedInstance].dispatchInterval = 20;
    
    // 4
    id<GAITracker> tracker = [[GAI sharedInstance] trackerWithTrackingId:@"UA-39787880-2"];
```
上面的代码首先把GAI的sharedInstance拿出来,然后往里面设置了一些属性,第一个是处理异常,第二个是设置log的等级,第三个设置了多少时间间隔往Google服务器上传一次数据,这里是20秒.第四个是使用tracking id新建一个tracker.

####Step 4.1. automatic screen tracking####

一个screen相当于app中的一个页面,或者说是view controller中的一个view,它可以用来track用户在一个view上停留了多长时间,还可以track用户在不同view中切换的动作.因为每个view都被一个view controller管理,所以为了实现automatic screen tracking,我们需要修改每个view的view controller.Google Analytics的SDK中提供了一个叫`GAITrackedViewController`的类,让我们自己的view controller继承它就行了,此外还需要修改这个类的一个内置属性`screenName`,它代表了当前这个被track的view的名字.

在我们的项目中修改`ClockViewController.h `,import并让它继承`GAITrackedViewController`,然后在`ClockViewController.m`的viewDidLoad方法中设置screenName属性:

```objective-c
#import <UIKit/UIKit.h>
#import "GAITrackedViewController.h"

@interface ClockViewController : GAITrackedViewController

@end
```

```objective-c
- (void)viewDidLoad {
    [super viewDidLoad];
    self.screenName = @"Clock";
	// Do any additional setup after loading the view, typically from a nib.
}
```

在上面的两部做完以后,我们就给这个view绑定了一个名字(Clock),在父类GAITrackedViewController中有一个viewWillAppear的方法,它会注册这个名叫Clock的view,此外在这个方法中还做了如下的几件事:

* 得到共有的tracker对象
* 告诉tracker去track名叫Clock的view(这里的view是Google Analytics中的概念,相当于一个屏幕)
* 创建日志
* 使用tracker往服务器上传数据.


####Step 4.2. manual screen tracking####

4.1中我们实现了自动track,但是它的作用有点局限,现在我们实现一个手动的screen tracking,它让我们对其他的用户行为track(例如点击按钮).

这一步我们在另一个类`StopWatchViewController.m`中做实验.打开这个类,然后在顶端加上如下引用:

```objective-c
#import GAIDictionaryBuilder.h
```

然后在`viewDidAppear`方法中加上下面的代码:

```objective-c
- (void)viewDidAppear:(BOOL)animated {
    [super viewDidAppear:animated];
    
    id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];
    [tracker set:kGAIScreenName value:@"Stopwatch"];
    [tracker send:[[GAIDictionaryBuilder createAppView] build]];
}
```

因为我们现在是手动配置,所以需要首先得到默认的tracker,然后设置这个view的screenName,最后一个是创建这个view record,然后send数据给server.之所以把这些步骤放到viewDIdAppear方法中,是因为我们想在每次用户方法这个view的时候都发送数据到server.

运行app,然后不断切换下面的tab(每一次到StopWatchViewController.m中都会调用viewDidAppear方法继而发送数据到server)

{% img /images/ga-10.png 300 600%}

最后再到Google Analytics的dashboard上看结果:

{% img /images/ga-9.png 600 1200%}


###Step 5. 其他配置###

接下来我们想要track在stopwatch上面的两个button,只需要做如下的事情:

* 在`StopWatchViewController.m`中创建一个logButtonPressed方法:

```objective-c
-(void)logButtonPress:(UIButton *)button{
    
    id<GAITracker> tracker = [[GAI sharedInstance] defaultTracker];
    
    [tracker set:kGAIScreenName value:@"Stopwatch"];
    [tracker send:[[GAIDictionaryBuilder createEventWithCategory:@"UX"
                                                          action:@"touch"
                                                           label:[button.titleLabel text]
                                                           value:nil] build]];
    [tracker set:kGAIScreenName value:nil];
}
```

上面的代码首先得到一个tracker,然后设置当前view的名字,因为这次我们不是记录用户浏览屏幕的行为,所以不像之前那样发送view到server,这次我们track的是用户点击按钮的事件,所以我们创建一个touch事件,这个事件被Google framework归类到了UX事件中,然后我们以被按的按钮的title作为事件的title然后传送到server上去.最后一步把screen name重设为nil是为了防止其他事件使用这个名字,因为tracker是共有的对象,在一个地方设置的state会影响另一个地方的使用,所以除了应该共享的state之外,在设置后应该还原.

* 在button对应的hooker中调用上面的方法. 

```objective-c
-(IBAction)startToggle:(id)sender{
[self logButtonPress:(UIButton *)sender];   
...
}

 -(IBAction)reset:(id)sender{
 [self logButtonPress:(UIButton *)sender];   
 ...
 }
```

运行app,不断点击start和reset按钮,然后来到dashboard上的Behavior->Events->Overview上看运行结果:

{% img /images/ga-11.png 600 1200%}

值得注意的是track的事件范围默认不包括今天,所以需要在右上角那里修改一下.


    

