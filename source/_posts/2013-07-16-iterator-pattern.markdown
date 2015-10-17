---
layout: post
title: "深入浅出设计模式之迭代器模式(Iterator Pattern)和组合模式(Composite Pattern)"
date: 2013-07-16 16:52
comments: true
categories: 
---
一、迭代器模式

迭代器模式应用了封装的原理，封装了可迭代组件的具体实现，游走于聚合内每一个元素，而又不暴露内部的表示。举个例子吧，有两家餐馆，它们现在合并了，这样提供的菜品就是原来的2倍。但是餐馆A使用数组来存储菜品，而餐馆B用列表来存储。如果想看所有的菜品，还得通过两种方式便利得到全部内容，这样很不方便，于是就想到了通过迭代器模式封装它们。
<!-- more -->
1、菜单接口，两家餐馆都实现了其中的迭代器创建方法
``` java
public interface Menu {
    public Iterator createIterator();
}
```
餐馆A
``` java
public class DinerMenu implements Menu{
    static final int MAX_ITEMS = 6;
    int numberOfItems = 0;

    MenuItem[] menuItems;

    public DinerMenu() {
        menuItems = new MenuItem[MAX_ITEMS];

        addItem("Vegetarian BLT", "Bacon with lettuce", true, 2.99);
        addItem("BLT", "Bacon with tomato", false, 2.99);
        addItem("Soup of the day", "With a side of potato salad", false, 3.49);
        addItem("hotdog", "hot dog with relish", false, 2.99);
    }

    private void addItem(String name, String description, boolean vegetarian, double price) {
        MenuItem item = new MenuItem(name, description, vegetarian, price);
        if (numberOfItems > MAX_ITEMS) {
            System.err.println("Sorry, menu is full");
        }
        menuItems[numberOfItems] = item;
        numberOfItems++;
    }

    @Override
    public Iterator createIterator() {
        return new DinerMenuIterator(menuItems);
    }
}
```
餐馆B
``` java
public class PancakeHouseMenu implements Menu{
    private List<MenuItem> menuItem;

    public PancakeHouseMenu() {
        this.menuItem = new ArrayList<MenuItem>();

        addItem("K&B's pancake breakfast", "Pancake with scrambled eggs, sausage", true, 2.99);
        addItem("Regular pancake breakfast", "Pancake with fried eggs, sausage", false, 2.99);
        addItem("Blueberry pancake", "Pancake made with fresh blueberries", true, 3.49);
        addItem("Wafles", "Wafles, with your choice of blueberries or strawberries", true, 3.49);
    }

    private void addItem(String name, String description, boolean vegetarian, double price) {
        MenuItem item = new MenuItem(name, description, vegetarian, price);
        menuItem.add(item);
    }

    @Override
    public Iterator createIterator() {
        return new PancakeHouseMenuIterator(menuItem);
    }
}
```
3、两家餐馆的迭代器实现
餐馆A的迭代器
``` java
public class DinerMenuIterator implements Iterator {
    private MenuItem[] menuItems;
    private int currentIndex;

    public DinerMenuIterator(MenuItem[] menuItems) {
        this.menuItems = menuItems;
        currentIndex = 0;
    }

    @Override
    public boolean hasNext() {
        return menuItems[currentIndex + 1] != null;
    }

    @Override
    public Object next() {
        MenuItem item = menuItems[currentIndex];
        currentIndex++;
        return item;
    }

    @Override
    public void remove() {
    }
}
```
餐馆B的迭代器
``` java
public class PancakeHouseMenuIterator implements Iterator {
    private List<MenuItem> pancakeHouseMenus;
    private int currentIndex;

    public PancakeHouseMenuIterator(List<MenuItem> pancakeHouseMenus) {
        this.pancakeHouseMenus = pancakeHouseMenus;
        currentIndex = 0;
    }

    @Override
    public boolean hasNext() {
        return pancakeHouseMenus.size() > currentIndex && pancakeHouseMenus.get(currentIndex) != null;
    }

    @Override
    public Object next() {
        MenuItem item = pancakeHouseMenus.get(currentIndex);
        currentIndex++;
        return item;
    }

    @Override
    public void remove() {
    }
}
```
我们可以看到，它们其实都实现自Iterator接口，这个接口是在java.util包中，相当于迭代器模式也是java内置的，我们只需要直接拿出来用就可以了。两个迭代器类提供给外界的接口都一样，所以客户并不需要知道具体实现，而只关心自己接收的类是否实现了Iterator就行了。
4、女服务员提供点菜功能，他需要打印给客户所有的菜品
``` java
public class Waitress {
    private PancakeHouseMenu pancakeHouseMenu;
    private DinerMenu dinerMenu;
    private CafeMenu cafeMenu;

    public Waitress(PancakeHouseMenu pancakeHouseMenu, DinerMenu dinerMenu, CafeMenu cafeMenu) {
        this.pancakeHouseMenu = pancakeHouseMenu;
        this.dinerMenu = dinerMenu;
        this.cafeMenu = cafeMenu;
    }

    public void printMenu() {
        System.out.println("MENU\n----\nBREAKFAST");
        printMenu(pancakeHouseMenu.createIterator());
        System.out.println("\nLUNCH");
        printMenu(dinerMenu.createIterator());
        System.out.println("\nCAFE TIME");
        printMenu(cafeMenu.createIterator());
    }

    public void printMenu(Iterator iterator) {
        while (iterator.hasNext()) {
            MenuItem menuItem = (MenuItem) iterator.next();
            System.out.println(menuItem.getName() + ", " + menuItem.getPrice() + " -- " + menuItem.getDescription());
        }
    }
}
```
女服务员拿到迭代器，她从菜单项的实现中解耦了。这样对她来说是很好的，因为她能使用同样的代码去遍历容易组能的元素。
5、开始点菜
``` java
public class WaitressAction {
    public static void main(String[] args) {
        PancakeHouseMenu pancakeHouseMenu = new PancakeHouseMenu();
        DinerMenu dinerMenu = new DinerMenu();
        CafeMenu cafeMenu = new CafeMenu();

        Waitress waitress = new Waitress(pancakeHouseMenu, dinerMenu, cafeMenu);
        waitress.printMenu();
    }
}
```

整个程序的构造如下图所示：
{% img /images/iterator_pattern-1.png %}

二、组合模式
组合模式的定义是允许将对象组合成树形结构来表现“整体/部分”层次结构。组合让客户以一致的方式去处理个别对象以及对象组合。例如，开始我们遍历的是菜单里的菜品，但是现在要求向菜单里加入子菜单，这样一个菜单里也许不光有菜品，还可能有子菜单。如果想要以一种相同的方式去处理菜品和菜单(也许是打印名称、描述)，就需要定义一个组件接口来作为菜单和菜单项的共同接口，这样就能够用统一的做法来处理菜单和菜单项。
1、统一接口
``` java
public abstract class MenuComponent {
    public void add(MenuComponent menuComponent) {
        throw new UnsupportedOperationException();
    }

    public void remove(MenuComponent menuComponent) {
        throw new UnsupportedOperationException();
    }

    public MenuComponent getChild(int i) {
        throw new UnsupportedOperationException();
    }

    public String getName() {
        throw new UnsupportedOperationException();
    }

    public String getDescription() {
        throw new UnsupportedOperationException();
    }

    public double getPrice() {
        throw new UnsupportedOperationException();
    }

    public boolean isVegetarian() {
        throw new UnsupportedOperationException();
    }

    public void print() {
        throw new UnsupportedOperationException();
    }

    public Iterator createIterator() {
        throw new UnsupportedOperationException();
    }
}
```
2、Menu(菜单)继承自统一接口
``` java
public class Menu2 extends MenuComponent {
    private List<MenuComponent> menuComponents = new ArrayList<MenuComponent>();
    private String name;
    private String description;

    public Menu2(String name, String description) {
        this.name = name;
        this.description = description;
    }

    public void add(MenuComponent component) {
        menuComponents.add(component);
    }

    public void remove(MenuComponent component) {
        menuComponents.remove(menuComponents);
    }

    public MenuComponent getChild(int index) {
        return menuComponents.get(index);
    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    public Iterator createIterator() {
        return new CompositeIterator(menuComponents.iterator());
    }

    public void print() {
        System.out.println("\n" + getName());
        System.out.println(", " + getDescription());
        System.out.println("----------------------");

        Iterator iterator = menuComponents.iterator();
        while (iterator.hasNext()) {
            MenuComponent component = (MenuComponent) iterator.next();
            component.print();
        }
    }
}
```
然后对应的DinerMenu和PancakeHouseMenu再继承Menu，这里就不写出来了。

3、(MenuItem)菜品继承统一接口
``` java
public class MenuItem extends MenuComponent {
    String name;
    String description;
    boolean vegetarian;
    double price;

    public MenuItem(String name, String description, boolean vegetarian, double price) {
        this.name = name;
        this.description = description;
        this.vegetarian = vegetarian;
        this.price = price;

    }

    public String getName() {
        return name;
    }

    public String getDescription() {
        return description;
    }

    public boolean isVegetarian() {
        return vegetarian;
    }

    public double getPrice() {
        return price;
    }

    public Iterator createIterator() {
        return new NullIterator();
    }
    public void print() {
        System.out.println(" " + getName());
        if (isVegetarian()) {
            System.out.print("(v)");
        }

        System.out.println(", " + getPrice());
        System.out.println("   -- " + getDescription());
    }
}
```
这样，菜单和菜品都会被统一对待。尽管他们由于继承，会得到一些不相干的方法，但是这个已经无关紧要。下面是结构图：

{% img /images/iterator_pattern-2.png %}

总而言之，当与偶数个对象的组合，他们彼此之间有“整体/部分”的关系，并且你想用一致的方式对待这些对象时，就需要用到组合模式。这样客户就不再需要操心面对的是组合对象还是叶节点对象。不必写一大堆if语句来保证他们对正确的对象调用了正确的方法。通常，他们只需要对整个结构调用一个方法并执行操作就可以了。

