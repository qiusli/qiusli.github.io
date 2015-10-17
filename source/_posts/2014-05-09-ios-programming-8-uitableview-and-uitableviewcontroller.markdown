---
layout: post
title: "iOS Programming 8 (UITableView &amp; UITableViewController)"
date: 2014-05-09 12:20
comments: true
categories: 
---
`UITableView`在iOS中是一个很重要的概念,它会把数据像列表一样显示在屏幕上.首先需要介绍几个概念:

* `UITableView`: 它是一个view,用于在屏幕上显示数据.它会向其对应的`dataSource`询问应该显示什么数据,然后再显示.对应的`dataShource`需要遵循`UITableViewDataSource`Protocol.
* `UITableView`同时需要一个delegate,用来处理在`UITableView`上传出来的事件.这个delegate需要遵循`UITableViewDelegate` Protocol.
* `UITableView`需要一个view controller来与之协作.

<!-- more -->

`UITableViewController`同时兼顾了view controller, delegate和data source这三个角色.`UITableViewController`继承自`UIViewController`,所以它有自己的view,这个view就是`UITableView`.`UITableViewController`负责`UITableView`的准备和显示.同时`UITableViewController`遵循了`UITableViewDataSource`和`UITableViewDelegate`Protocol,所以它既是data source又是delegate.

`UITableView` `UITableViewController`对应的关系如下:

 {% img /images/ios-8_1.png%}
 
首先创建一个`UITableViewController`的子类.

```objective-c
@interface BNRItemsViewController : UITableViewController
@end
```

`UITableViewController`默认的初始化方法是`initWithStyle:`,这里的style可以是`UITableViewStylePlain`或者`UITableViewStyleGrouped`,不过在iOS7里面,两者的差别并不大.为了传给它一个style,我们可以让这个默认初始化方法调用另一个init方法,然后再到里面去调用默认的初始化方法:

```objective-c
- (instancetype) init {
    self = [super initWithStyle:UITableViewStylePlain];
    return self;
}

- (instancetype) initWithStyle:(UITableViewStyle)style {
    return [self init];
}
```

这个时候我们只是建立了一个`UITableViewController`,但是并没有把它加到view hierachy里面去,所以我们来到AppDelegate的`application:didFinishLaunchingWithOptions`方法,在里面初始化这个`BNRItemsViewController`并把它作为window的root controller.

```objective-c
BNRItemsViewController *itemsViewController = [[BNRItemsViewController alloc] init];
self.window.rootViewController = itemsViewController;
```

现在运行,可以看到已经有table view的效果了,但是里面没有数据

 {% img /images/ios-8_2.png 300 600%}
 
接下来我们就应该往里面加数据.需要复用在前几章建立的BNRItem类,这个类代表了具体的商品.接下来还需要建立一个BNRItemStore类,我们的tableview controller会从这个store里面获取数据.具体怎么建立的这里就不阐述了,总之这个store有`createItem`和`allItems`两个方法,前者用来创建item后者用来获取所有的item用来显示.现在我们可以创建50个item用于显示:

```objective-c
- (instancetype) init {
    self = [super initWithStyle:UITableViewStylePlain];
    BNRItemStore *store = [BNRItemStore sharedStore];
    for (int i = 0; i < 50; i++) {
        [store createItem];
    }
    return self;
}
```

当`UITableView`需要显示的时候,它会向data source发送一些消息,然后通过这些回馈判断应该显示多少行数据,每行都显示什么等等.`UITableViewController`有两个必须实现的方法,这两个方法被定义在了Protocol中,它们分别是`tableView:numberOfRowsInSection`和`tableView:cellForRowAtIndexPath`,前者是用来得到需要显示多少行,后者是用来告诉每行显示的数据的具体内容.因为我们一共建立了50条数据,所以可以用前面的方法来得到行数:

```objective-c
- (NSInteger) tableView:(UITableView *)tableView numberOfRowsInSection:(NSInteger)section {
    return [[[BNRItemStore sharedStore] allItems] count];
}
```

在介绍后者之前,需要插入解释一个概念 -- `UITableViewCell`,这个对象对应于table中每一行的数据,即table中的每一行都由一个`UITableViewCell`填充.这个对象有一个subview名叫`contentView`,它是用来具体显示数据的,以文本或图片的形式.这个`contentView`里有三个property:`detailTextLabel` `textLabel` `imageView`,今天我们将会用到第二个.

下面是方法的实现:

```objective-c
- (UITableViewCell *) tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    UITableViewCell *tableViewCell = [[UITableViewCell alloc] initWithStyle:UITableViewCellStyleDefault reuseIdentifier:@"UITableViewCell"];
    
    NSArray *items = [[BNRItemStore sharedStore] allItems];
    tableViewCell.textLabel.text = [[items objectAtIndex:indexPath.row] description];

    return tableViewCell;
}
```

首先建立一个cell,然后得到所有items,并根据传进来的indexPath获取到特定的item,然后把item的description赋给cell的textLabel.这个方法是在每一行需要显示的时候被调用,所以indexPath对应于table view中对应的行数.

再次运行,发现tableview被填充了数据:

 {% img /images/ios-8_3.png 300 600%}
 
有时候,由于需要显示的内容非常多,如果使用上面的方法需要每次都建立tableviewcell,这样非常耗时耗内存,一个解决方法是建立特定数量的cell把它们放到一个cell pool里,然后当下滑tableview的时候,从pool里面提取cell来显示数据,把刚刚消失的cell放回到pool里去.为了达到上面的效果,我们只需要做两处改变:

* 注册cell类
* 在`tableView:cellForRowAtIndexPath`方法里面提取

我们可以在`BNRTableViewController`的`viewDidLoad`方法里注册:

```objective-c
- (void) viewDidLoad {
    [super viewDidLoad];
    [self.tableView registerClass:[UITableViewCell class] forCellReuseIdentifier:@"UITableViewCell"];
}
```

然后修改`tableView:cellForRowAtIndexPath`方法:

```objective-c
- (UITableViewCell *) tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    UITableViewCell *tableViewCell = [tableView dequeueReusableCellWithIdentifier:@"UITableViewCell" forIndexPath:indexPath];
    
    NSArray *items = [[BNRItemStore sharedStore] allItems];
    tableViewCell.textLabel.text = [[items objectAtIndex:indexPath.row] description];

    return tableViewCell;
}
```

这里有一个identifier参数叫做UITableViewCell,这是因为我们通常在pool里面会放着各种不同类型的cell,在注册和提取的时候都给identifier标上名相当于给它们贴了标签,想要什么类型的cell只需要在提取的时候带上同样的标签就可以了.