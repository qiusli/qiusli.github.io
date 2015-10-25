---
layout: post
title: "JavaScript中的继承"
date: 2015-10-25 16:30:11 -0600
comments: true
categories: 科技
---
由于Javascript中的每个对象都有一个prototye,它也是对象的一个property.这个prototype也有自己的prototype,它能继承自那个prototype,而当前对象也能从自己的prototype那里继承property,这样就形成了一个prototype chain,这就是Javascript中继承的实现方式.简单来说,我们可以把当前对象的prototype(原型)看成是一个模板,当前对象继承自那个模板,然后扩充自己的方法.当前对象从其prototype那里获得一些property,而当前对象的prototype从其自身的prototype(prototype's prototype)那里也获得一些property.每一个通过Object literal或者new Object这种方式创建出的对象,其prototype指向object.prototype,意味着当前对象能获得object.prototype里的property.

``` Javascript
var book = {
    title: "red and black"
};
var p = Object.getPrototypeOf(book);
console.log(p == Object.prototype);   // true
```
所有对象都从Object.prototype那里继承了一些方法,最常见的有5个:`hasOwnProperty`, `propertyIsEnumerable`,`isPrototypeOf`,`valueOf`,`toString`.

实现继承最简单的方式就是显式指定当前对象的prototype(原型),这样它就能从其原型那里获得方法.我们可以使用Object.create()方法来实现这个目的,它会新创建一个对象,然后这个对象可以被赋值给一个变量.create第一个参数指定了谁是当前对象的原型,第二个参数是可选的,意味着一些自定义的property和其属性.

``` Javascript
// specify reference of prototypr of current object
var person1 = {name: "Qiushi"};
// is same as
var person2 = Object.create(Object.prototype, {
    name: {
        enumerable: true,
        configurable: true,
        writable: true,
        value: "Qiushi"
    }
});
```

``` Javascript
// inherit from other objects
var company = {
    name: "Goldman Sachs",
    sayName: function () {
        console.log("Company name is " + this.name)
    }
};
// anotherCompany自己定义了name property,它覆盖了从company那里继承来的name property.
var anotherCompany = Object.create(company, {
    name: {
        enumerable: true,
        configurable: true,
        writable: true,
        value: "ThoughtWorks"
    }
});
company.sayName();    // Company name is Goldman Sachs
anotherCompany.sayName();  // Company name is ThoughtWorks
console.log(company.hasOwnProperty("sayName")); // true
console.log(anotherCompany.hasOwnProperty("sayName")); // false
console.log(company.isPrototypeOf(anotherCompany)); // true
console.log(Object.getPrototypeOf(anotherCompany) == company); // true
var thirdCompany = Object.create(anotherCompany, {
    name: {
        configurable: true,
        writable: true,
        enumerable: true,
        value: "ThinkGeek"
    }
});
console.log(Object.getPrototypeOf(thirdCompany)); // { name: 'ThoughtWorks' }
```
下面这张图解释了原型链:

{% img /images/JS_inheritance/1.png %}

当一个方法被调用时,系统会首先去当前对象的own property里寻找,如果找到了就返回,否则回去当前对象的prototype里寻找,如果找到就返回,否则继续去当前对象的prototype的prototype里继续寻找,这个寻找链的终结是Obejct.prototype.

有时候我们会想创建一个不继承自任何原型的对象,它在一些场景下很实用,例如被当做hash来用.

``` Javascript
var nakedObject = Object.create(null);console.log("toString" in nakedObject);     // falseconsole.log("valueOf" in nakedObject);      // false
```
结合上面的create方法,同时我们知道每个方法都有自己的prototype,当我们在定义一个方法时,系统为我们指定了当前方法的prototype,同时指定了prototype里constructor的指向,具体如下:

``` Javascript
function YourConstructor() {
    // initialization
}
// JavaScript engine does this for you behind the scenes
YourConstructor.prototype = Object.create(Object.prototype, {
    constructor: {
        configurable: true,
        enumerable: true,
        value: YourConstructor,
        writable: true
    }
});
```

这其实是系统创建了一个Obejct.prototype的对象,然后让当前方法的prototype指向它.
下面我们来看一个复杂一些的例子:

``` Javascript
// overwrite prototype property
function Rectangle(length, width) {
    this.length = length;
    this.width = width
}
Rectangle.prototype.getArea = function() {
    return this.length * this.width;
};
Rectangle.prototype.toString = function() {
    return "[Rectangle " + this.length + "x" + this.width + "]";
};

function Square(size) {
    // constructor stealing, 调用父类的构造器
    Rectangle.call(this, size, size);
}
Square.prototype = new Rectangle();
Square.prototype.constructor = Square;
Square.prototype.toString = function() {
    return "[Square " + this.length + "x" + this.width + "]";
};
var rect = new Rectangle(5, 10);
var square = new Square(6);
console.log(rect.getArea());               // 50
console.log(square.getArea());             // 36

console.log(rect.toString());              // [Rectangle 5x10]
console.log(square.toString());            // [Square 6x6]

console.log(rect instanceof Rectangle);    // true
console.log(rect instanceof Object);       // true
console.log(square instanceof Square);     // true
console.log(square instanceof Rectangle);  // true
console.log(square instanceof Object);     // true
```
Square的对象同时是Square, Rectangle和Object的对像,下面这张图解释了原因:
{% img /images/JS_inheritance/2.png %}