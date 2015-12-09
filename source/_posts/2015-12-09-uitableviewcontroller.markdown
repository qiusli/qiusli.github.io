---
layout: post
title: "浅析UITableViewController"
date: 2015-12-09 02:08:29 -0700
comments: true
categories: Tech
---
UITableViewController是iOS开发中最常用到的viewController类,它可以一行行地显示数据.首先介绍一些重要概念.
##cell##
cell用来显示在每一行上的数据,它不是table中的行,而是一个view.可以把它想象成覆盖在每一行上的一个画板,如果我们想要在每一行上显示数据,首先应该得到这一行的cell,然后设置cell中的一些属性.在Xcode中可以选择cell的种类,不同的种类有不同的格局,例如一些cell类型有title,另一些还支持image等.

除了系统自带的cell的属性,我们还能往这些cell上添加自己的一些属性,例如Text Field等.为了能通过cell得到这些属性,我们可以给它们设置一个数字tag,然后在得到cell后通过下面的代码来得到我们新添加的属性:

```swift
let label = cell.viewWithTag(1000) as! UILabel
```

我们不会为每一行创建一个cell,相反一般只会创建一个prototype cell,然后再反复重用这个cell(需要给这个cell一个identifier,让系统能通过其找到它).如果当前手机屏幕只能显示10行,那么系统最多会创建11个cell,多出来的那个cell用于当用户上下滑动时,新的行进入了当前显示屏幕,但是某个旧的行没有完全退出屏幕,所以此时需要额外的一个cell.用户在滑动时退出屏幕的cell被回收,然后等待被新进入屏幕的行使用. 
##delegate##
(viewController指代在storyboard上显示的scene,viewController**类**指代其对应的swift文件)

为了避免让viewController的逻辑太过复杂,iOS开发中使用了delegate来把viewController中的一些职责分离出去,让其他一些viewController类(通常是当前viewController对应的类)来帮助当前viewController做一些事情,这个辅助类叫做当前viewController的delegate.在UITableViewController中,为了给tableView提供数据,我们需要让其对应的viewController类实现data source的delagate,这样这个viewController类就拥有了提供数据的能力,然后通过实现一些预订的方法就能提供数据了. 在iOS开发中,我们无需另去实现data source的delegate,因为如果某个viewController类继承自UITableViewController,它自动被赋予了提供数据的能力,同时也被赋予了响应用户操作的能力.

```swift
class ChecklistViewController: UITableViewController
```

viewController类给viewController提供数据是一个被动的过程,只在viewController想要取得数据的时候才会去联系其delegate(对应的viewController类),通常是调用这个delegate里预订的方法,然后delegate返回数据给viewController,一次调用就结束了.


####*作为数据源*####
最常用的两个方法是`tableView(numberOfRowsInSection)`和`tableView(cellForRowAtIndexPath)`, viewController通过前者得知有多少行需要显示,通过后者得到每一行的cell,然后我们可以在这个cell上做文章.

```swift
override func tableView(tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
    return 5
}
    
override func tableView(tableView: UITableView, cellForRowAtIndexPath indexPath: NSIndexPath) -> UITableViewCell {
    let cell = tableView.dequeueReusableCellWithIdentifier("ChecklistItem", forIndexPath: indexPath)
    let item = checklist.items[indexPath.row]
    configureTextForCell(cell, withChecklistItem: item)
    configureCheckmarkForCell(cell, withChecklistItem: item)
        
    return cell
}
```
(上面第一行通过`tableView.dequeueReusableCellWithIdentifier("ChecklistItem", forIndexPath: indexPath)`来为某一个indexPath上行得到/创建cell.有3点值得注意,其一是tableView对象,它是内置对象,只要一个viewController类继承自UITableViewController他就会获得这个对象.其二是indexPath,它是一个对象,封装了section和row.其三是tableView.dequeueReusableCellWithIdentifier方法,它通过一个identifier试图去寻找空余的cell来给当前行使用,如果没有找到就创建一个新的cell)


####*作为某些行为的响应*####
用户可以在tableViewController上做一些操作,例如点击某一行,类似行为的响应方法通常都定义在delegate中.

```swift
override func tableView(tableView: UITableView, didSelectRowAtIndexPath indexPath: NSIndexPath) {
    if let cell = tableView.cellForRowAtIndexPath(indexPath) {
        let item = checklist.items[indexPath.row]
        item.toggle()
        configureCheckmarkForCell(cell, withChecklistItem: item)
    }
    tableView.deselectRowAtIndexPath(indexPath, animated: true)
}
```
(上面通过`tableView.cellForRowAtIndexPath`得到某个indexPath上的cell)
这样viewController在接受到这些用户行为后会向其delegate发出请求,然后delegate来具体实现某些功能,然后把结果返回给viewController.

总而言之,为了实现代理模式,我们需要做以下操作:
1. 让某个viewController**类**实现代理(接口)
2. 实现接口中定义的方法
3. 告诉viewController哪个类是其代理