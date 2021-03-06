---
title: 面向对象编程-构造函数与 new 命令
date: 2015-10-01 10:08:34
tags: -面向对象编程
---
# 目录
+ 对象的概念
+ 构造函数
+ new 命令
  - 基本用法
  - new命令的原理
  - new.target
虽然不同于传统的面向对象编程语言，但是JavaScript具有很强的面向对象编程能力。本章介绍JavaScript如何进行“面向对象编程”。

## 1.对象的概念
“面向对象编程”（Object Oriented Programming，缩写为OOP）是目前主流的编程范式。它的核心思想是将真实世界中各种复杂的关系，抽象为一个个对象，然后由对象之间的分工与合作，完成对真实世界的模拟。
<!-- more -->
传统的过程式编程（procedural programming）由一系列函数或一系列指令组成，而面向对象编程的程序由一系列对象组成。每一个对象都是功能中心，具有明确分工，可以完成接受信息、处理数据、发出信息等任务。因此，面向对象编程具有灵活性、代码的可重用性、模块性等特点，容易维护和开发，非常适合多人合作的大型软件项目。

那么，“对象”（object）到底是什么？

我们从两个层次来理解。

（1）“对象”是单个实物的抽象。

一本书、一辆汽车、一个人都可以是“对象”，一个数据库、一张网页、一个与远程服务器的连接也可以是“对象”。当实物被抽象成“对象”，实物之间的关系就变成了“对象”之间的关系，从而就可以模拟现实情况，针对“对象”进行编程。

（2）“对象”是一个容器，封装了“属性”（property）和“方法”（method）。

所谓“属性”，就是对象的状态；所谓“方法”，就是对象的行为（完成某种任务）。比如，我们可以把动物抽象为animal对象，“属性”记录具体是那一种动物，“方法”表示动物的某种行为（奔跑、捕猎、休息等等）。

## 2.构造函数
“面向对象编程”的第一步，就是要生成“对象”。

前面说过，“对象”是单个实物的抽象。通常需要一个模板，表示某一类实物的共同特征，然后“对象”根据这个模板生成。

典型的面向对象编程语言（比如 C++ 和 Java），存在“类”（class）这个概念。所谓“类”就是对象的模板，对象就是“类”的实例。但是，JavaScript语言的对象体系，不是基于“类”的，而是基于构造函数（constructor）和原型链（prototype）。

JavaScript语言使用构造函数（constructor）作为对象的模板。所谓“构造函数”，就是专门用来生成“对象”的函数。它提供模板，描述对象的基本结构。一个构造函数，可以生成多个对象，这些对象都有相同的结构。

构造函数的写法就是一个普通的函数，但是有自己的特征和用法。
```
var Vehicle = function () {
  this.price = 1000;
};
```
上面代码中，Vehicle就是构造函数，它提供模板，用来生成对象实例。为了与普通函数区别，构造函数名字的第一个字母通常大写。

构造函数的特点有两个。

+ 函数体内部使用了this关键字，代表了所要生成的对象实例。
+ 生成对象的时候，必需用new命令，调用Vehicle函数。

## 3.new 命令
### 3.1基本用法
new命令的作用，就是执行构造函数，返回一个实例对象。
```
var Vehicle = function (){
  this.price = 1000;
};

var v = new Vehicle();
v.price // 1000
```
上面代码通过new命令，让构造函数Vehicle生成一个实例对象，保存在变量v中。这个新生成的实例对象，从构造函数Vehicle继承了price属性。在new命令执行时，构造函数内部的this，就代表了新生成的实例对象，this.price表示实例对象有一个price属性，它的值是1000。

使用new命令时，根据需要，构造函数也可以接受参数。
```
var Vehicle = function (p) {
  this.price = p;
};

var v = new Vehicle(500);
```
new命令本身就可以执行构造函数，所以后面的构造函数可以带括号，也可以不带括号。下面两行代码是等价的。
```
var v = new Vehicle();
var v = new Vehicle;
```
一个很自然的问题是，如果忘了使用new命令，直接调用构造函数会发生什么事？

这种情况下，构造函数就变成了普通函数，并不会生成实例对象。而且由于后面会说到的原因，this这时代表全局对象，将造成一些意想不到的结果。
```
var Vehicle = function (){
  this.price = 1000;
};

var v = Vehicle();
v.price
// Uncaught TypeError: Cannot read property 'price' of undefined

price
// 1000
```
上面代码中，调用Vehicle构造函数时，忘了加上new命令。结果，price属性变成了全局变量，而变量v变成了undefined。

因此，应该非常小心，避免出现不使用new命令、直接调用构造函数的情况。为了保证构造函数必须与new命令一起使用，一个解决办法是，在构造函数内部使用严格模式，即第一行加上use strict。
```
function Fubar(foo, bar){
  'use strict';
  this._foo = foo;
  this._bar = bar;
}

Fubar()
// TypeError: Cannot set property '_foo' of undefined
```
上面代码的Fubar为构造函数，use strict命令保证了该函数在严格模式下运行。由于在严格模式中，函数内部的this不能指向全局对象，默认等于undefined，导致不加new调用会报错（JavaScript不允许对undefined添加属性）。

另一个解决办法，是在构造函数内部判断是否使用new命令，如果发现没有使用，则直接返回一个实例对象。

```
function Fubar(foo, bar){
  if (!(this instanceof Fubar)) {
    return new Fubar(foo, bar);
  }

  this._foo = foo;
  this._bar = bar;
}

Fubar(1, 2)._foo // 1
(new Fubar(1, 2))._foo // 1
```
上面代码中的构造函数，不管加不加new命令，都会得到同样的结果。

### 3.2new命令的原理
使用new命令时，它后面的函数调用就不是正常的调用，而是依次执行下面的步骤。

+ 创建一个空对象，作为将要返回的对象实例
+ 将这个空对象的原型，指向构造函数的prototype属性
+ 将这个空对象赋值给函数内部的this关键字
+ 开始执行构造函数内部的代码

也就是说，构造函数内部，this指的是一个新生成的空对象，所有针对this的操作，都会发生在这个空对象上。构造函数之所以叫“构造函数”，就是说这个函数的目的，就是操作一个空对象（即this对象），将其“构造”为需要的样子。

如果构造函数内部有return语句，而且return后面跟着一个对象，new命令会返回return语句指定的对象；否则，就会不管return语句，返回this对象。
```
var Vehicle = function () {
  this.price = 1000;
  return 1000;
};

(new Vehicle()) === 1000
// false
```
上面代码中，构造函数Vehicle的return语句返回一个数值。这时，new命令就会忽略这个return语句，返回“构造”后的this对象。

但是，如果return语句返回的是一个跟this无关的新对象，new命令会返回这个新对象，而不是this对象。这一点需要特别引起注意。
```
var Vehicle = function (){
  this.price = 1000;
  return { price: 2000 };
};

(new Vehicle()).price
// 2000
```
上面代码中，构造函数Vehicle的return语句，返回的是一个新对象。new命令会返回这个对象，而不是this对象。

另一方面，如果对普通函数（内部没有this关键字的函数）使用new命令，则会返回一个空对象。
```
function getMessage() {
  return 'this is a message';
}

var msg = new getMessage();

msg // {}
typeof msg // "Object"
```
上面代码中，getMessage是一个普通函数，返回一个字符串。对它使用new命令，会得到一个空对象。这是因为new命令总是返回一个对象，要么是实例对象，要么是return语句指定的对象。本例中，return语句返回的是字符串，所以new命令就忽略了该语句。

new命令简化的内部流程，可以用下面的代码表示。
```
function _new(/* 构造函数 */ constructor, /* 构造函数参数 */ param1) {
  // 将 arguments 对象转为数组
  var args = [].slice.call(arguments);
  // 取出构造函数
  var constructor = args.shift();
  // 创建一个空对象，继承构造函数的 prototype 属性
  var context = Object.create(constructor.prototype);
  // 执行构造函数
  var result = constructor.apply(context, args);
  // 如果返回结果是对象，就直接返回，则返回 context 对象
  return (typeof result === 'object' && result != null) ? result : context;
}

// 实例
var actor = _new(Person, '张三', 28);
```
### 3.3new.target
函数内部可以使用new.target属性。如果当前函数是new命令调用，new.target指向当前函数，否则为undefined。
```
function f() {
  console.log(new.target === f);
}

f() // false
new f() // true
```