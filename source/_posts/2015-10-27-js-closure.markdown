---
layout: post
title: "JavaScript中的闭包"
date: 2015-10-27 19:02:20 -0600
comments: true
categories: 科技
---
##为什么要用闭包##
在JavaScript中没有私有变量的概念，即使一个变量定义在方法或者对象内部，在外面也仍然能够直接访问，闭包的出现解决了这个问题。

##什么是闭包##
简单地说，闭包就是定义在一个函数内部的函数，它能访问其父级函数里定义的变量。比如下面这个例子

``` JavaScript
function outer() {
    var i = 1;

    function inner() {
        i++;
        return i;
    }

    return inner;
}
var k = outer();       
k();
console.log(k()); // 3
```
inner定义在outer内部，它能访问定义在outer中的变量i，outer返回inner函数的引用。变量k接受outer函数执行后的返回值（即inner），然后执行一次，i增加了1，最后再执行了一次，最后被打印出来。我们看到在outer执行完并返回之后，定义在outer内部的变量i并没有随之消逝，而是一直存在于内存中直到整个程序结束。

``` Javascript
function sayName(firstname) {
    var x = "Full name: ";

    function inner(lastName) {
        return x + firstname + " " + lastName;
    }

    return inner;
}
var a = sayName("Qiushi");
console.log(a("Li"));      // Full name: Qiushi Li
```
上面的程序第一次调用sayName的时候传入了firstname，然后x和firstname都被记录在内存中，然后inner函数被返回并被a引用。下一次执行a的时候，其实是执行了inner函数，这时lastname被传入，结合之前记录的结合起来形成了最终的结果。

使用闭包实现getter和setter并隐藏被操作的变量：

``` Javascript
function modify() {
    var i = 100;
    return {
        set: function (k) {
            i = k;
        },
        get: function () {
            return i;
        }
    };
}
var m = modify();
console.log(m.get()); // 100
m.set(103); 
console.log(m.get()); // 103
```

##闭包的原理##
我们都知道一个变量或者函数都有其作用域和定义域，它们不能脱离定义其的环境。例如一个定义在函数中的变量，当这个函数执行完毕并返回结果之后，定义在其中的变量也会被系统回收。类似于Java，Javavascript也有垃圾回收机制，可以概括为:
1. 如果一个对象没有被其他对象指向，它会被GC回收
2. 如果两个对象互相指向，同时不再被其他对象指向，那么这2个对象都会被回收

回到闭包的原理上来，当我们把内部函数返回时，它通常会被外部的一个变量引用，这时系统不能回收内部函数，同时因为这个内部函数依赖于定义其的外部函数而存在，所以这个内部函数的外部环境也不能被回收，所以定义在其中的变量也一直常驻于内存。虽然这个变量在内存中，但是我们不能通过除了内部函数访问之外的方式去访问它，因为外部函数已经执行完毕了。

``` Javascript
var obj = (function(){
    var a = 1;
    function increase1(){
        a++;
        console.log(a);
    }
    function increase2(){
        a++;
        console.log(a);
    }
    return {
        b:increase1,            
        c:increase2
    }
})();
obj.b();      //2
obj.c();      //3
```
上面的例子的意思是，首先定义一个外部函数，它里面有两个内部函数，返回一个有两个property的对象，这两个property分别是对两个内部函数的引用。这个外部函数在定义完成之后被立即执行，所以obj指向的是包含两个property的对象。然后obj调用那个对象的property，访问property指向的内部函数。

##写闭包常犯的错##
我们来看下面这段代码：

``` Javascript
function addId(dogArray) {
    var i;
    var commonPre = 100;
    for (i = 0; i < dogArray.length; i++) {
        dogArray[i]["id"] = function () {
            return commonPre + i;
        }
    }
}
var dogArray = [
    {name: "Ted", id: new Function()},
    {name: "Snoop", id: new Function()},
    {name: "Husky", id: new Function()}
];
addId(dogArray);
var ted = dogArray[0];
console.log(ted["id"]()); // 103
```
预计输出的结果应该是100，为什么是103呢？原因久在于在for循环时，每个dogArray元素的id property被赋值了一个方法，即指向了那个方法，但是那个方法并没有立即执行，而是在试图打印的时候才开始执行。同时那个方法中存放的是对i的引用，即for循环中的i，而不是i的值，所以在for循环执行完成之后i的值是3，所以该内部函数在执行的时候i等于3，得到最终结果为103.

怎么修改呢？其实很简单，就是在定义该内部函数的地方立即执行它，此时i的引用的值就是当前i的值。

``` Javascript
// modified
function addIdBetter(dogArray) {
    var i;
    var commonPre = 100;
    for (i = 0; i < dogArray.length; i++) {
        dogArray[i]["id"] = (function () {
            return commonPre + i;
        })();
    }
}
addIdBetter(dogArray);
var snoop = dogArray[1];
console.log(snoop["id"]); // 101
```

##闭包的潜在危害##
因为外部函数在其内部函数被引用时不能被释放，所以会增加内存的消耗。

简而言之，闭包的出现使Javascript隐藏内部变量成为可能，他是Javascript中的难点，也是其特色之一。