---
layout: post
title: "JavaScript中的构造器(constructor)和原型(prototype)"
date: 2015-10-24 22:22:16 -0600
comments: true
categories: 科技
---
##构造器##
JavaScript中的构造器其实就是方法/函数,我们新建一个方法,然后使用new去创建对象,可以说这个方法就是被创建的对象的原型.如果一个方法被作为对象的原型,约定俗成它应该大写字母开头.

``` Javascript
function Person(name) {
    this.name = name;
    this.sayName = function() {
        console.log(this.name);
    };
}
var person1 = new Person("Nick");
var person2 = new Person("Craig");
console.log(person1 instanceof  Person);    // true
console.log(person2 instanceof  Person);    // true
```
如果方法不接受参数,在使用new创建的时候可以省去括号. 在通过函数创建对象的时候,函数里的this自动指向创建出来的对象.

``` Javascript
function Person(){...}
var person1 = new Person;
```
每个新建的对象都有一个constructor property,它指向创建这个对象的方法.如果这个对象是使用Obejct literal或者new Object方法创建的,那么该对象的constructor指向object. 这里需要指出的是在使用person1.constructor的时候由于constructor不是person1的ownproperty,所以系统会自动去创建这个instance的function里的prototype里去找,即Person里的一个名字叫做prototype的arribute,它可能的定义是:
```JavaScript
function Person(name) {
    this.name = name;
    this.sayName = function() {
        console.log(this.name);
    };
    this.prototype = {
    	constructor: Person,
    	toString: function() {console.log('xxx');}
    	...
    }
}
```

``` JavaScript
console.log(person1.constructor == Person); // true
console.log(person2.constructor == Person); // true
```
在使用构造器的时候前面要加上new关键字,表示被赋值的是一个对象,否则只是简单的方法调用.

``` Javascript
var person1 = Person("Nicholas");         // note: missing "new"console.log(person1 instanceof Person);   // falseconsole.log(typeof person1);              // "undefined"console.log(name);                        // "Nicholas"
```

这里需要专门所以说函数和对象的区别,函数的定义是function(){},对象的定义是var obj = {}或者var obj = new Object().对象里的内容是键值对{key: value},函数里的内容和Java的构造函数里的内容类似function(name){this.name = name}. 函数可以用来创建对象.

##原型##
在创建对象时,我们有时会有这样一个疑虑:既然一些方法处理的事情是一样的,不过接受的参数不一样,我们为什么不能让许多对象共用这些方法呢?这就是prototype的由来, 和Java里的static有点像.

每个对象都有一个prototype,prototype自身也是一个对象,它也有自己的property.几乎每一个函数都有自己的prototype property,所有通过该方法创建的对象都能访问该函数的propotype.我们可以把一个对象的原型想象成它的一个模板,这个对象从它的模板原型中得到一些property,然后基于这个继续扩展.

``` Javascript

var book = {
    title: "the principle of object oriented javascript"
};
console.log("hasOwnProperty" in book);                            // true
console.log(book.hasOwnProperty("hasOwnProperty"));               // false
console.log(Object.prototype.hasOwnProperty("hasOwnProperty"));   // true
```
每一个对象在被创建后都有自己的[[Prototype]]property,它指向创建它的方法的prototype.

 {% img /images/JS_prototype/1.png %}

当在试图访问一个对象上得某个property时,系统会先查看当前对象是否有这个own property,如果有则返回,否则继续查找它的[[Prototype]]指向的对象(即创建这个对象的方法的property)是否有这个property,有则返回否则返回undefined.prototype property不能被复写或者删除:

``` Javascript
var object = {};
console.log(object.toString());  // [object Object]
object.toString = function() {
    return "[object Custom]";
};
console.log(object.toString());  // [object Custom]
delete object.toString;
console.log(object.toString());  // [object Object]
delete object.toString;
console.log(object.toString());  // [object Object]
```
上面的代码首先打印原型的toString方法,然后试图修改创建这个object的方法的prototype(Object的prototype),第二次成功打印出修改后的值.其实它并不是修改了Object里的prototype,而是创建了自己的一个own property,这个own property和prototype中的toString有相同的函数签名.然后我们试图删除prototype里的toString方法,这只是删除了自己定义的own property,最后一次是真正尝试删除prototype里的toString方法,但是并不成功,因为它不能在对象上删除.

 {% img /images/JS_prototype/2.png %}
 
###创建prototype###

``` Javascript
function Car(brand) {
    this.brand = brand;
}
Car.prototype.sayBrand = function() {
    console.log(this.brand);
};
var car1 = new Car("BMW");
var car2 = new Car("Benz");
car1.sayBrand();                       // BMW
car2.sayBrand();                       // Benz
```
因为prototype是被所有对象共用的,这也许会产生一些潜在问题,即一个对象对prototype做出的改变会影响到另一个对象.

``` Javascript
console.log("5. ------");
Car.prototype.manufacureLocation = [];
car1.manufacureLocation.push("Chongqing");
car2.manufacureLocation.push("Beijing");
console.log(car1.manufacureLocation);      // [ 'Chongqing', 'Beijing' ]
console.log(car2.manufacureLocation);      // [ 'Chongqing', 'Beijing' ]
```

###同时创建多个prototype的property###
``` Javascript
function University(name) {
    this.name = name
}
University.prototype = {
    sayName: function() {
        console.log(this.name)
    },
    toString: function() {
        console.log("WUSTL")
    }
};
```
这样做很方便,但是它也有一个潜在的问题,即复写了方法自身的prototype.之前,在方法被创建时,它在prototype里有自己的constructor,它指向方法自身.但是在上面的University.prototype = {...}语句其实是使用Object literal方法对University方法的prototype重新赋值了,所以它的prototype里的constructor指向了新的对象的prototype,即object得prototype.

``` Javascript
var univ = new University("Washington University");
console.log(univ instanceof University);     // true
console.log(univ.constructor == University); // false
console.log(univ.constructor == Object);     // true
```
为了避免这种情况的出现,我们可以显式制定方法的constructor的指向:

``` Javascript
// solution to the above issue
function Computer(brand) {
    this.brand = brand
}
Computer.prototype = {
    constructor: Computer,
    sayBrand: function() {
        console.log("brand is " + this.brand)
    },
    statePrice: function() {
        console.log("price is quite high")
    }
};
var computer = new Computer("Lenovo");
console.log(computer instanceof Computer);     // true
console.log(computer.constructor == Computer); // true
```

``` Javascript
var computer1 = new Computer("Apple");
var computer2 = new Computer("Dell");
Computer.prototype.quote = function(quote) {
    console.log("quote of " + this.brand + " is " + quote)
};
computer1.quote("Think Different"); // quote of Apple is Think Different
computer2.quote("Whatever");        // quote of Dell is Whatever
```