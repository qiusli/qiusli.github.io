---
layout: post
categories: 科技
date: "2015-10-22 19:21:12 -0600"
comments: true
title: JavaScript中的函数
---

JavaScript中的函数即对象,它也是以键值对的形式存在,但是它内部有一个internal
property叫做[[call]],它区分了其和对象的唯一不同,定义了自身的类型是函数,可以被执行调用,而对象只能被创建使用.同时如果使用typeof来查看类型,会返回function,internal
property从外部不可达.

<!-- more -->

#### 两种定义方法

``` Javascript
// function declaration
function add(a, b) {
    return a + b;
}

// function expression
var k = function(a, b) {
    return a + b;
};
```

这两种定义方法基本类似,唯一的区别就是使用前者定义的方法可以在任何地方调用,因为系统总会把定义自动放到最前面(function
hoist),后者不会.

方法在Javascript中是一等公民,意味着你可以把它赋值给变量,把它加到方法里,作为参数传递,作为返回值等等.

``` Javascript
// pass function as parameter
var numbers = [3, 6, 2, 8, 4, 1, 7, 6, 8];
numbers.sort(function(first, second) {
    return first - second;
});
console.log(numbers);  // [ 1, 2, 3, 4, 6, 6, 7, 8, 8 ]
```

#### 参数

函数可以接受零个或多个参数,函数接受的所有参数都被存在一个叫做arguments的变量中,可以在方法内部用其取出传递进来的参数.

``` Javascript
function sum() {
    var i = 0, sum = 0;
    for(; i < arguments.length; i++) {
        sum += arguments[i];
    }
    return sum;
}
console.log(sum(1, 2, 3));     // 6
console.log(sum(1, 2, 3, 4));  // 10
```

上面的例子sum函数没有定义接受的参数个数,因为无论多少个,最终都是被存放在arguments变量里.如果定义了有限个的参数,多余的参数会被忽略.

``` Javascript
function refect(value) {
    return value;
}
console.log(refect("hi"));           // hi
console.log(refect("hi", 25));       // 25
console.log(refect("hi", 22, true)); // hi
```

由于函数在JavaScript中等同于对象,所有函数也可以有自己的属性(property).length就是其中的一个属性,它记录了函数接受的参数个数.

``` Javascript
function refect(value) {
    return value;
}
console.log(refect.length); // 1
```

#### 多态

多态的区分是在函数签名,具体是方法名和参数的组合来区分.但是因为JavaScript函数能接受任意多的参数,所以导致不能够用参数来区分多个多态方法,所以JavaScript中没有多态一说.

``` Javascript
function sayMessage(message) {
    console.log(message);
}

function sayMessage() {
    console.log("Default Message");
}
sayMessage("Chelse");   // Default Message
```

在JavaScript中,如果你定义了多个同名方法,那么在你调用时,系统会取最后一个方法,前面的同名方法都被shadow掉了.因为函数等同于对象,系统在解析上面的代码时,其实是完成了下面的操作:

``` Javascript
var sayMessage = new Function("message", "console.log(message);");
// 变量指向了另一个对象
sayMessage = new Function("console.log(\"Default Message chongqing\");")
sayMessage("chongqing"); 
```

#### 关键字: this

JavaScript代码中每一个scope里都有一个this关键字,代表了调用当前函数的那个对象.

``` Javascript
var person = {
    name: "Qiushi",
    sayName: function() {
        console.log(this.name);
    }
};
function sayNameForAll() {
    console.log(this.name);
}
var person1 = {
    name: "chelse",
    sayName: sayNameForAll
};
var person2 = {
    name: "ruth",
    sayName: sayNameForAll
};
// 这里的this代表person对象.
person.sayName();
// 这里的this代表person1对象.
person1.sayName();
// 这里的this代表person2对象.
person2.sayName();
name = "shawnee";
// 这里的this代表全局对象,即window,所以window.name只能是shawnee.
sayNameForAll();
```

上面的this指向都是隐式的,我们可以通过下面三种方法来显式制定this的指向.

#### 使用call

``` Javascript
function speak(label) {
    console.log(label + this.language);
}
var p1 = {
    language: "chinese"
};
var p2 = {
    language: "spanish"
};
speak.call(p1, "我说 ");     // 我说 chinese
speak.call(p2, "I speak "); // I speak spanish
```

call方法的第一个参数指定this的指向,后面为参数.

#### 使用apply

``` Javascript
speak.apply(p1, ["Kevin speaks "]); // Kevin speaks chinese
```

apply和call的唯一区别就是接受的第二个参数的类型,call接受单个参数,而apply接受一个数组,所以如果定义的方法接受多个参数,最好使用apply,反之则使用call.

#### 使用bind

``` Javascript
function anotherSayName(label) {
    console.log(label + this.name);
}
var n1 = {
    name: "Ani"
};
var n2 = {
    name: "Rashmi"
};
var sayName1 = anotherSayName.bind(n1, "n1");
sayName1();                                           // n1Ani
var sayName2 = anotherSayName.bind(n2, "n2");
sayName2();                                       // n2Rashmi
n2.sayName = sayName1;
n2.sayName();                                     // n1Ani
```

值得注意的是最后输出为n1Ani而不是n2Rashmi,因为sayName1在最开始就指向了anotherSayName,this指向了n1,在重新赋值给n2的sayName时,其this指向不会被改变,所以输出n1Ani.
