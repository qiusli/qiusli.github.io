---
layout: post
title: "浅析UINavigationController"
date: 2015-12-16 20:27:15 -0700
comments: true
categories: tech 
---
##基本原理##
UINavigationController是一个容器，它提供基本的视图之间跳转的功能。viewcontroller被它包装起来，每一个viewController有自己的navigation item，当用户在左右滑动的时候，其实就是UINavigationController在起作用，它把另一个viewController放进屏幕替代当前屏幕上的viewController，同时这个viewController有自己的Navigation Item，这个item有回退键和title。UINavigationController提供一个UINavigation bar，这个bar用来显示当前屏幕上的viewController的Navigation Item。

##如何把当前viewController放进UINavigationController##
进入storyboard，选中当前viewController，然后在状态栏上选择editor -> embed in -> Navigation Controller。之后，所有指向这个viewController的指针都指向了包装它的Navigation Controller。

##如何在不同的viewController之间跳转##
在上一步完成之后，这个viewController就有了一个Navigation Controller提供的Navigation bar，我们可以在这个Navigation Bar上添加一些Bar Button，然后ctrl＋drag这个按钮到其他viewController上，松开手时系统会提示你一些segue的选择。iOS 9提供了4种跳转方式:show, show detail, present modally, present as popover。如果选择show，当点击按钮跳转到另一个viewController时，它会有Navigation bar，并且还有一个back按钮，点击后回到之前的页面。如果选在present modally，点击按钮跳转后，页面上没有navigation bar，自然也没有回退按钮，通常我们为了让这个viewController有Navigation Bar，我们需要把它再次包装到另一个Navigation Controller里，最后的效果如下:

{% img /images/navigationcontroller-1.png %}

##如何在不同viewController之间传递数据##
例如A，B之间跳转，我们可以在B中定义delegate和方法，然后让A实现delegate和方法，同时B中保存一个变量，它指向A。viewController有一个方法叫做`prepareForSegue`，它在两个controller跳转之前被调用，我们可以在里面放上准备数据，例如:

```swift
override func prepareForSegue(segue: UIStoryboardSegue, sender: AnyObject?) {
    if segue.identifier == "AddItem" {
        let navContr = segue.destinationViewController as? UINavigationController
        let contr = navContr?.topViewController as? AddItemViewController
        contr?.delegate = self
    } else if segue.identifier == "EditItem" {
        let navContr = segue.destinationViewController as? UINavigationController
        let contr = navContr?.topViewController as? AddItemViewController
        contr?.delegate = self
        
        if let indexPath = tableView.indexPathForCell(sender as! UITableViewCell) {
            contr?.itemToEdit = items[indexPath.row]
        }
    }
}
```

因为一个controller可以有很多种方法跳到另一个controller(tap button, tap row...)，所以可以有很多segue在两个controller之间，我们需要给每一个segue一个别名来区别它们，上面的代码的意思是，如果segue是AddItem，然后就找到这个segue的目标controller，因为下一个controller被Navigation Controller包装，所以不是直接跳转而是通过包装Navigation Controller中转，得到Navigation Controller后找到它的topViewController，即最终目标，然后设置它的delegate变量为当前controller，这样就能在B中通过delegate变量调用A中实现的delegate方法，达到操作A中数据的目的。

B中定义的delegate和变量:

```swift
protocol AddItemViewControllerDelegate: class {
    func addItemViewControllerDidCancel(controller: AddItemViewController)
    func addItemViewController(controller: AddItemViewController, didFinishAddingItem item: ChecklistItem)
    func addItemViewController(controller: AddItemViewController, didFinishEditingItem item: ChecklistItem)
}

class AddItemViewController {
	...
	var itemToEdit: ChecklistItem?
    ...
}
```

A中实现delegate和其方法:

```swift
class ChecklistViewController: AddItemViewControllerDelegate {
	...
	func addItemViewController(controller: AddItemViewController, didFinishAddingItem item: ChecklistItem) {
    let indexRow = items.count
    items.append(item)
    let indexPath = NSIndexPath(forRow: indexRow, inSection: 0)
    tableView.insertRowsAtIndexPaths([indexPath], withRowAnimation: .Automatic)
    dismissViewControllerAnimated(true, completion: nil)
}
func addItemViewController(controller: AddItemViewController, didFinishEditingItem item: ChecklistItem) {
    if let indexRow = items.indexOf(item) {
        let indexPath = NSIndexPath(forRow: indexRow, inSection: 0)
        if let cell = tableView.cellForRowAtIndexPath(indexPath) {
            configureTextForCell(cell, item: item)
        }
    }
    dismissViewControllerAnimated(true, completion: nil)
}
	...
}
```