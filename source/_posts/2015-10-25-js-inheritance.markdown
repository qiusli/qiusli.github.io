---
layout: post
title: "JavaScript中的继承"
date: 2015-10-25 16:30:11 -0600
comments: true
categories: 科技
---
由于Javascript中的每个对象都有一个[[prototye]],它是当前对象的一个property(就像下面那个例子里的title一样,而这个prototype的存在可能是如下),
``` Javascript
var book = {
    title: "red and black",
    prototype: {
    	key: value,
    	...
    }
};
```

由于每个对象都有自己的propotype,在创建当前对象时,此对象的prototype自动指向创建它的funciton的prototpye属性,这样就形成了一个prototype chain,这就是Javascript中继承的实现方式.意味着当前对象能使用创建其的函数的prototype里的东西,例如上面我创建了book对象,我可以使用book.toString()来打印对象,但是toStirng并不是book对象自身拥有的方法,所以这时系统会到Object.prototype里去找toString方法.

``` Javascript
var book = {
    title: "red and black"
};
var p = Object.getPrototypeOf(book);
console.log(p == Object.prototype);   // true
```
所有对象都从Object.prototype那里继承了一些方法,最常见的有5个:`hasOwnProperty`, `propertyIsEnumerable`,`isPrototypeOf`,`valueOf`,`toString`.

实现继承最简单的方式就是显式指定当前对象的prototype,这样它就能从那里获得方法.我们可以使用Object.create()方法来实现这个目的,它会新创建一个对象,然后这个对象可以被赋值给一个变量.create第一个参数指定了谁是当前对象的prototype指向哪里,第二个参数是可选的,意味着一些自定义的property和其属性.

``` Javascript
// specify reference of prototypr of current object
var person1 = {name: "Qiushi"};
// is same as
var person1 = Object.create(Object.prototype, {
    name: {
        enumerable: true,
        configurable: true,
        writable: true,
        value: "Qiushi"
    }
});
```

person1对象的prototype指向Object.prototype,这时系统默认的指向方式,但是我们可以改变当前对象的prototype的指向:

``` Javascript
// inherit from other objects
var person1 = {
    name: "Nick",
    sayName: function () {
        console.log("My name is " + this.name)
    }
};
// anotherCompany自己定义了name property,它覆盖了从company那里继承来的name property.
var person2 = Object.create(person1, {
    name: {
        enumerable: true,
        configurable: true,
        writable: true,
        value: "Jeff"
    }
});
person1.sayName();    // My name is Nick
person2.sayName();  // My name is Jeff
console.log(person1.hasOwnProperty("sayName")); // true
console.log(person2.hasOwnProperty("sayName")); // false
console.log(person1.isPrototypeOf(person2)); // true
console.log(Object.getPrototypeOf(person2) == person1); // true
```

这里值得注意的是,在创建person2时,它的prototype指向了person1对象本身,而不是person1对象的prototype.通常我们在使用var obj = new Book()的时候,obj指向Book函数的prototype,而上面我们显示地指定了让person的prototype指向person1对象自身.

下面这张图解释了原型链:

{% img /images/JS_inheritance/1.png %}

当一个方法被调用时,系统会首先去当前对象的own property里寻找,如果找到了就返回,否则回去当前对象的prototype里寻找,如果找到就返回,否则继续去当前对象的prototype指向的对象里继续寻找(如果没有显示指定prototype的指向,一般是指向创建当前对象的函数.否则就到显示指定的那个对象里去找),这个寻找链的终结是Obejct.prototype.

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
Square.prototype = new Rectangle(); /*explanation below*/
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

上面在注释了/*explanation below*/那一行是说,Square的prototype是通过Rectangle创建,所以Square.prototype里找不到的方法回到Rectangle.prototype里去找.因为Square.prototype是通过Rectangle函数创建,下面这张图解释了原因:
{% img /images/JS_inheritance/2.png %}