---
layout: post
title: "iOS Programming 1(A simple iOS application)"
date: 2014-03-06 14:07
comments: true
categories: 
---
最近在学习iOS开发，虽然课上也有讲，但是还是不如自己看书学得更实在，今天就从最基础的iOS开发说起。值得注意的是，我在这类博客里所讲到的内容，完全是按照iOS Programming: The Big Nerd Ranch Guide第四版里面来的。
<!-- more -->
项目最后出来的效果类似于下面这张图：
	
{% img /images/ios1.png 300 600%}
	
首先新建一个empty application，此时的项目里面只有WUSTLAppDelegate文件。今天我们的目的是根据MVC模式建立一个简单的项目，所以，第一步我们从controller出发，建立一个名为WUSTLQuizViewController的类，需要注意的是，在建立的时候一定要勾上'with XIB for user interface'这个选项，这样系统会给你建一个后缀名为xib的文件。其实我们常常把xib文件和Interface Builder搞混，以为他俩是一回事，其实不然。Interface Builder其实是一个对象的编辑器，你可以在里面创建和修改对象(例如button和label)，然后把他们保存到一个文件中，这个保存下来的文件就是xib文件，这就不难理解为什么xib文件的全称是XML Interface Builder了。

接下来我们应该创建MVC中的View部分了，我们分别拖动两个label和两个button到interface builder里面，然后调整他们的大小等，以达到上图的效果，这样就算完成了View部分。但是现在的View只是一个独立的东西，它不能传递信息给controller，也不能从controller得到显示的指令，所以下一个我们需要做的就是把controller和view连接起来。连接使两个在内存中的对象互相知道了对方，这样他们就能互相交流了。在Interface Builder中有两种连接可以使用：Outlet和Action。Outlet指向一个对象，Action是用来响应view中事件的具体方法。
	
View里面有四个对象，所以我们也需要在相应的controller里面建立四个对象，在WUSTLQuizViewController.m里将@implementation和@end之间系统自动填充的代码全部删除，然后再在WUSTLQuizViewController.m的@interface和@end之间(category)加上如下代码(首先我们只加label部分)：

```objective-c
    @property (nonatomic, weak) IBOutlet UILabel *questionLabel;
	@property (nonatomic, weak) IBOutlet UILabel *answerLabel;
```

这两行代码的意思是，在每个WUSTLQuizViewController.m的对象中建立两个IBOutlet对象，分别叫做questionLabel和answerLabel，他们都指向UILabel，关键字IBOutlet的作用是告诉Interface Builder我们将用这个被Iboutlet修饰的对象连接Interface Builder中的一个对象。

既然Controller和View两边都声明了对象，剩下就工作就是把他们连接起来了。我们找到xib文件，然后找到左边Placeholders，右键点击后会出现下面这样的弹框让你选择：
	
{% img /images/ios2.png 400 400 %}

我们可以看到在小黑框里面有我们刚刚在controller里面生命的Outlet对象，左键点击小黑框中变量右边的加号，然后连出一条线指向Label，如果Label高亮了就松开，这就创建了一个controller中的Outlet对象和View中的label对象的连接。
	
{% img /images/ios3.png 600 600%}

同样的事再做一遍，连接另一个label。值得注意的是，刚才我们连线的时候是从小黑框到Interface Builder，这个顺序是有讲究的，因为我们想把在controller中的Outlet在View中显示，所以就是从小黑框到Interface Builder这个指向。

UILabel已经连接了，那么接下来应该连接button了。因为我们想要的效果是在用户点击button的时候View给controller发送消息，这个时候在controller中应该有处理这个消息的方法，所以我们到WUSTLQuizViewController.m中建立两个对应的方法:

```objective-c
	- (IBAction)showQuestion:(id)sender {
	}

	- (IBAction)showAnswer:(id)sender {
	}
```

上面代码中IBAction是告诉Xcode，这个方法是用来响应Interface Builder中的事件的。说到这里，不得不提一下Target\Action模式，他是objective-c中的一种消息机制，当用户在view中做出一些动作后，view会发送消息到controller，Target指的就是响应这个事件的对象(在这里就是WUSTLQuizViewController.m)，Action指的是Target中的哪个方法具体用来处理这个事件的(在这里Action分别是showQuestion和showAnswer方法)。这时我们再回到xib文件中，点击Interface Builder中的button，然后按住不放拉出一条线并在Placeholder中的file‘s owner松开(control-drag)：

{% img /images/ios4.png 600 600%}
	
对两个button分别做同样的动作，我们就完成了button到controller的连接，也就是两次Target/Action操作。上面的file’s owner对于与xib文件来说就是controller，所以下面在做连接操作的时候，实际上就是View中的对象和Controller里面的对象做连接。接下来我们就需要回到controller中去实现具体的响应操作了。我们在WUSTLQuizViewController.m中刚才加label的地方加上两个NSArray和一个index，NSArray用来装question和answer，index用来存当前NSArray的下标。
	
```objective-c
	#import "WUSTLQuizViewController.h"

	@interface WUSTLQuizViewController ()
	@property (nonatomic) int currentQuestionIndex;

	@property (nonatomic, copy) NSArray *questions;
	@property (nonatomic, copy) NSArray *answers;

	@property (nonatomic, weak) IBOutlet UILabel *questionLabel;
	@property (nonatomic, weak) IBOutlet UILabel *answerLabel;
	@end
	
	@implementation
	...
	@end
```

这里需要提到另一个方法:

```objective-c
    initWithNibName:bundle:
```

这个方法是在包含他的controller被创建的时候调用的，我们可以在里面做一些初始化，例如在这里我们可以在里面初始化两个NSArray，如下：
```objective-c
	- (instancetype) initWithNibName:(NSString *)nibNameOrNil bundle:(NSBundle 			*)nibBundleOrNil {
    	// call the init method implemented by the superclass
	    self = [super initWithNibName:nibNameOrNil bundle:nibBundleOrNil];
    
    	if (self) {
        	// create two arrays filled with questions and answers
	        // and make the pointers point to them
    	    self.questions = @[@"From what is cognac made?",
    	     @"What is 7 + 7?", 
    	     @"What is the captal of Vermont?"];
    	    self.answers = @[@"Grapes", @"14", @"Montpelier"];
    	}
    
	    // return the address of the new object
    	return self;
	}
```
那么这里的nib又是什么呢？注意到之前我们用到了xib文件，其实nib文件就是xib文件编译后产生的，他比xib文件更小，并且对于应用来说更容易解析。每一个应用都有一个叫做application bundle的文件夹，这里面专门存放这个有用会使用到的可执行文件和一些其他的资源，nib文件也会在编译完成后被拷贝到这里面。在运行时，系统会载入application bundle里面的资源并在需要的时候使用。
	
我们接下来实现showQuestion和showAnswer方法了。由于在之前已经把他俩和Interface Builder中的button关联起来了，所以button在被点之后，这两个方法会相应地被调用。我们想要在show question点击的时候在label中显示问题，所以可以这么实现：

```objective-c
- (IBAction)showQuestion:(id)sender {    
    self.currentQuestionIndex++;
    
    if (self.currentQuestionIndex == [self.questions count]) {    
        self.currentQuestionIndex = 0;
    }
    
    NSString *question = self.questions[self.currentQuestionIndex];
    self.questionLabel.text = question;
    self.answerLabel.text = @"???";
}
```

同理，现实答案的button可以这么实现：

```objective-c
- (IBAction)showAnswer:(id)sender {
    NSString *answer = self.answers[self.currentQuestionIndex];
    self.answerLabel.text = answer;
}
```
controller和view已经连接完毕了，但是还没有完，如果这个时候启动项目会发现屏幕上什么都没有，因为没有把view加入到显示中去。我们需要做的就是在AppDelegate的application方法中把当前的controller作为整个应用的root view：

```objective-c
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary 	*)launchOptions {  
    self.window = [[UIWindow alloc] initWithFrame:[[UIScreen mainScreen] bounds]];
	// Override point for customization after application launch.
    
    WUSTLQuizViewController *quizVC = [[WUSTLQuizViewController alloc] init];
    self.window.rootViewController = quizVC;
    
    self.window.backgroundColor = [UIColor whiteColor];
    [self.window makeKeyAndVisible];
    return YES;
}
```

每个iOS应用都有一个AppDelegate，他是应用的入口，当应用启动和关闭的时候，可以在AppDelegate里面相应的方法中做初始化和收尾的工作。
	
到这个时候，每次启动应用的时候，一个controller对象会被创建，然后initWithNibName方法会被调用，所有准备工作结束，可以启动项目了。