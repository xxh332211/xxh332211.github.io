---
title: 面向对象编程-this 关键字
date: 2015-10-6 10:27:00
tags: -面向对象编程
---
# 目录
+ 涵义
+ 使用场合
+ 使用注意点
+ 绑定 this 的方法
    - function.prototype.call()
    - function.prototype.apply()
    - function.prototype.bind()
+ 参考链接
## 1.涵义
this关键字是一个非常重要的语法点。毫不夸张地说，不理解它的含义，大部分开发任务都无法完成。

首先，this总是返回一个对象，简单说，就是返回属性或方法“当前”所在的对象。
```
this.property
```
上面代码中，this就代表property属性当前所在的对象。
<!-- more -->
下面是一个实际的例子。
```
var person = {
  name: '张三',
  describe: function () {
    return '姓名：'+ this.name;
  }
};

person.describe()
// "姓名：张三"
```
上面代码中，this.name表示describe方法所在的当前对象的name属性。调用person.describe方法时，describe方法所在的当前对象是person，所以就是调用person.name。

由于对象的属性可以赋给另一个对象，所以属性所在的当前对象是可变的，即this的指向是可变的。
```
var A = {
  name: '张三',
  describe: function () {
    return '姓名：'+ this.name;
  }
};

var B = {
  name: '李四'
};

B.describe = A.describe;
B.describe()
// "姓名：李四"
```
上面代码中，A.describe属性被赋给B，于是B.describe就表示describe方法所在的当前对象是B，所以this.name就指向B.name。

稍稍重构这个例子，this的动态指向就能看得更清楚。
```
function f() {
  return '姓名：'+ this.name;
}

var A = {
  name: '张三',
  describe: f
};

var B = {
  name: '李四',
  describe: f
};

A.describe() // "姓名：张三"
B.describe() // "姓名：李四"
```
上面代码中，函数f内部使用了this关键字，随着f所在的对象不同，this的指向也不同。

只要函数被赋给另一个变量，this的指向就会变。
```
var A = {
  name: '张三',
  describe: function () {
    return '姓名：'+ this.name;
  }
};

var name = '李四';
var f = A.describe;
f() // "姓名：李四"
```
上面代码中，A.describe被赋值给变量f，内部的this就会指向f运行时所在的对象（本例是顶层对象）。

再看一个网页编程的例子。
```
<input type="text" name="age" size=3 onChange="validate(this, 18, 99);">

<script>
function validate(obj, lowval, hival){
  if ((obj.value < lowval) || (obj.value > hival))
    console.log('Invalid Value!');
}
</script>
```
上面代码是一个文本输入框，每当用户输入一个值，就会调用onChange回调函数，验证这个值是否在指定范围。回调函数传入this，就代表传入当前对象（即文本框），然后就可以从this.value上面读到用户的输入值。

总结一下，JavaScript语言之中，一切皆对象，运行环境也是对象，所以函数都是在某个对象之中运行，this就是这个对象（环境）。这本来并不会让用户糊涂，但是JavaScript支持运行环境动态切换，也就是说，this的指向是动态的，没有办法事先确定到底指向哪个对象，这才是最让初学者感到困惑的地方。

如果一个函数在全局环境中运行，那么this就是指顶层对象（浏览器中为window对象）。
```
function f() {
  return this;
}

f() === window // true
```
上面代码中，函数f在全局环境运行，它内部的this就指向顶层对象window。

可以近似地认为，this是所有函数运行时的一个隐藏参数，指向函数的运行环境。

## 2.使用场合
this的使用可以分成以下几个场合。

（1）全局环境

在全局环境使用this，它指的就是顶层对象window。
```
this === window // true

function f() {
  console.log(this === window); // true
}
```
上面代码说明，不管是不是在函数内部，只要是在全局环境下运行，this就是指顶层对象window。

（2）构造函数

构造函数中的this，指的是实例对象。
```
var Obj = function (p) {
  this.p = p;
};

Obj.prototype.m = function() {
  return this.p;
};
```
上面代码定义了一个构造函数Obj。由于this指向实例对象，所以在构造函数内部定义this.p，就相当于定义实例对象有一个p属性；然后m方法可以返回这个p属性。
```
var o = new Obj('Hello World!');

o.p // "Hello World!"
o.m() // "Hello World!"
```
（3）对象的方法

当A对象的方法被赋予B对象，该方法中的this就从指向A对象变成了指向B对象。所以要特别小心，将某个对象的方法赋值给另一个对象，会改变this的指向。

请看下面的代码。
```
var obj ={
  foo: function () {
    console.log(this);
  }
};

obj.foo() // obj
```
上面代码中，obj.foo方法执行时，它内部的this指向obj。

但是，只有这一种用法（直接在obj对象上调用foo方法），this指向obj；其他用法时，this都指向代码块当前所在对象（浏览器为window对象）。
```
// 情况一
(obj.foo = obj.foo)() // window

// 情况二
(false || obj.foo)() // window

// 情况三
(1, obj.foo)() // window
```
上面代码中，obj.foo先运算再执行，即使它的值根本没有变化，this也不再指向obj了。

可以这样理解，在JavaScript引擎内部，obj和obj.foo储存在两个内存地址，简称为M1和M2。只有obj.foo()这样调用时，是从M1调用M2，因此this指向obj。但是，上面三种情况，都是直接取出M2进行运算，然后就在全局环境执行运算结果（还是M2），因此this指向全局环境。

上面三种情况等同于下面的代码。
```
// 情况一
(obj.foo = function () {
  console.log(this);
})()

// 情况二
(false || function () {
  console.log(this);
})()

// 情况三
(1, function () {
  console.log(this);
})()
```
同样的，如果某个方法位于多层对象的内部，这时为了简化书写，把该方法赋值给一个变量，往往会得到意料之外的结果。
```
var a = {
  b: {
    m: function() {
      console.log(this.p);
    },
    p: 'Hello'
  }
};

var hello = a.b.m;
hello() // undefined
```
上面代码中，m是多层对象内部的一个方法。为求简便，将其赋值给hello变量，结果调用时，this指向了顶层对象。为了避免这个问题，可以只将m所在的对象赋值给hello，这样调用时，this的指向就不会变。
```
var hello = a.b;
hello.m() // Hello
```
（4）Node

在Node中，this的指向又分成两种情况。全局环境中，this指向全局对象global；模块环境中，this指向module.exports。
```
// 全局环境
this === global // true

// 模块环境
this === module.exports // true
```
## 3.使用注意点
（1）避免多层this

由于this的指向是不确定的，所以切勿在函数中包含多层的this。
```
var o = {
  f1: function () {
    console.log(this);
    var f2 = function () {
      console.log(this);
    }();
  }
}

o.f1()
// Object
// Window
```
上面代码包含两层this，结果运行后，第一层指向该对象，第二层指向全局对象。实际执行的是下面的代码。
```
var temp = function () {
  console.log(this);
};

var o = {
  f1: function () {
    console.log(this);
    var f2 = temp();
  }
}
```
一个解决方法是在第二层改用一个指向外层this的变量。
```
var o = {
  f1: function() {
    console.log(this);
    var that = this;
    var f2 = function() {
      console.log(that);
    }();
  }
}

o.f1()
// Object
// Object
```
上面代码定义了变量that，固定指向外层的this，然后在内层使用that，就不会发生this指向的改变。

事实上，使用一个变量固定this的值，然后内层函数调用这个变量，是非常常见的做法，有大量应用，请务必掌握。

JavaScript 提供了严格模式，也可以硬性避免这种问题。在严格模式下，如果函数内部的this指向顶层对象，就会报错。
```
var counter = {
  count: 0
};
counter.inc = function () {
  'use strict';
  this.count++
};
var f = counter.inc;
f()
// TypeError: Cannot read property 'count' of undefined
```
上面代码中，inc方法通过'use strict'声明采用严格模式，这时内部的this一旦指向顶层对象，就会报错。

（2）避免数组处理方法中的this

数组的map和foreach方法，允许提供一个函数作为参数。这个函数内部不应该使用this。
```
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    this.p.forEach(function (item) {
      console.log(this.v + ' ' + item);
    });
  }
}

o.f()
// undefined a1
// undefined a2
```
上面代码中，foreach方法的回调函数中的this，其实是指向window对象，因此取不到o.v的值。原因跟上一段的多层this是一样的，就是内层的this不指向外部，而指向顶层对象。

解决这个问题的一种方法，是使用中间变量。
```
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    var that = this;
    this.p.forEach(function (item) {
      console.log(that.v+' '+item);
    });
  }
}

o.f()
// hello a1
// hello a2
```
另一种方法是将this当作foreach方法的第二个参数，固定它的运行环境。
```
var o = {
  v: 'hello',
  p: [ 'a1', 'a2' ],
  f: function f() {
    this.p.forEach(function (item) {
      console.log(this.v + ' ' + item);
    }, this);
  }
}

o.f()
// hello a1
// hello a2
```
（3）避免回调函数中的this

回调函数中的this往往会改变指向，最好避免使用。
```
var o = new Object();

o.f = function () {
  console.log(this === o);
}

o.f() // true
```
上面代码表示，如果调用o对象的f方法，其中的this就是指向o对象。

但是，如果将f方法指定给某个按钮的click事件，this的指向就变了。
```
$('#button').on('click', o.f);
```
点击按钮以后，控制台会显示false。原因是此时this不再指向o对象，而是指向按钮的DOM对象，因为f方法是在按钮对象的环境中被调用的。这种细微的差别，很容易在编程中忽视，导致难以察觉的错误。

为了解决这个问题，可以采用下面的一些方法对this进行绑定，也就是使得this固定指向某个对象，减少不确定性。

## 4.绑定 this 的方法
this的动态切换，固然为JavaScript创造了巨大的灵活性，但也使得编程变得困难和模糊。有时，需要把this固定下来，避免出现意想不到的情况。JavaScript提供了call、apply、bind这三个方法，来切换/固定this的指向。

### 4.1function.prototype.call()
函数实例的call方法，可以指定函数内部this的指向（即函数执行时所在的作用域），然后在所指定的作用域中，调用该函数。
```
var obj = {};

var f = function () {
  return this;
};

f() === this // true
f.call(obj) === obj // true
```
上面代码中，在全局环境运行函数f时，this指向全局环境；call方法可以改变this的指向，指定this指向对象obj，然后在对象obj的作用域中运行函数f。

call方法的参数，应该是一个对象。如果参数为空、null和undefined，则默认传入全局对象。
```
var n = 123;
var obj = { n: 456 };

function a() {
  console.log(this.n);
}

a.call() // 123
a.call(null) // 123
a.call(undefined) // 123
a.call(window) // 123
a.call(obj) // 456
```
上面代码中，a函数中的this关键字，如果指向全局对象，返回结果为123。如果使用call方法将this关键字指向obj对象，返回结果为456。可以看到，如果call方法没有参数，或者参数为null或undefined，则等同于指向全局对象。

如果call方法的参数是一个原始值，那么这个原始值会自动转成对应的包装对象，然后传入call方法。
```
var f = function () {
  return this;
};

f.call(5)
// Number {[[PrimitiveValue]]: 5}
```
上面代码中，call的参数为5，不是对象，会被自动转成包装对象（Number的实例），绑定f内部的this。

call方法还可以接受多个参数。
```
func.call(thisValue, arg1, arg2, ...)
```
call的第一个参数就是this所要指向的那个对象，后面的参数则是函数调用时所需的参数。
```
function add(a, b) {
  return a + b;
}

add.call(this, 1, 2) // 3
```
上面代码中，call方法指定函数add内部的this绑定当前环境（对象），并且参数为1和2，因此函数add运行后得到3。

call方法的一个应用是调用对象的原生方法。
```
var obj = {};
obj.hasOwnProperty('toString') // false

// 覆盖掉继承的 hasOwnProperty 方法
obj.hasOwnProperty = function () {
  return true;
};
obj.hasOwnProperty('toString') // true

Object.prototype.hasOwnProperty.call(obj, 'toString') // false
```
上面代码中，hasOwnProperty是obj对象继承的方法，如果这个方法一旦被覆盖，就不会得到正确结果。call方法可以解决这个方法，它将hasOwnProperty方法的原始定义放到obj对象上执行，这样无论obj上有没有同名方法，都不会影响结果。

### 4.2function.prototype.apply()
apply方法的作用与call方法类似，也是改变this指向，然后再调用该函数。唯一的区别就是，它接收一个数组作为函数执行时的参数，使用格式如下。
```
func.apply(thisValue, [arg1, arg2, ...])
```
apply方法的第一个参数也是this所要指向的那个对象，如果设为null或undefined，则等同于指定全局对象。第二个参数则是一个数组，该数组的所有成员依次作为参数，传入原函数。原函数的参数，在call方法中必须一个个添加，但是在apply方法中，必须以数组形式添加。

请看下面的例子。
```
function f(x,y){
  console.log(x+y);
}

f.call(null,1,1) // 2
f.apply(null,[1,1]) // 2
```
上面的f函数本来接受两个参数，使用apply方法以后，就变成可以接受一个数组作为参数。

利用这一点，可以做一些有趣的应用。

（1）找出数组最大元素

JavaScript不提供找出数组最大元素的函数。结合使用apply方法和Math.max方法，就可以返回数组的最大元素。
```
var a = [10, 2, 4, 15, 9];

Math.max.apply(null, a)
// 15
```
（2）将数组的空元素变为undefined

通过apply方法，利用Array构造函数将数组的空元素变成undefined。
```
Array.apply(null, ["a",,"b"])
// [ 'a', undefined, 'b' ]
```
空元素与undefined的差别在于，数组的forEach方法会跳过空元素，但是不会跳过undefined。因此，遍历内部元素的时候，会得到不同的结果。
```
var a = ['a', , 'b'];

function print(i) {
  console.log(i);
}

a.forEach(print)
// a
// b

Array.apply(null, a).forEach(print)
// a
// undefined
// b
```
（3）转换类似数组的对象

另外，利用数组对象的slice方法，可以将一个类似数组的对象（比如arguments对象）转为真正的数组。
```
Array.prototype.slice.apply({0:1,length:1})
// [1]

Array.prototype.slice.apply({0:1})
// []

Array.prototype.slice.apply({0:1,length:2})
// [1, undefined]

Array.prototype.slice.apply({length:1})
// [undefined]
```
上面代码的apply方法的参数都是对象，但是返回结果都是数组，这就起到了将对象转成数组的目的。从上面代码可以看到，这个方法起作用的前提是，被处理的对象必须有length属性，以及相对应的数字键。

（4）绑定回调函数的对象

上一节按钮点击事件的例子，可以改写成
```
var o = new Object();

o.f = function () {
  console.log(this === o);
}

var f = function (){
  o.f.apply(o);
  // 或者 o.f.call(o);
};

$('#button').on('click', f);
```
点击按钮以后，控制台将会显示true。由于apply方法（或者call方法）不仅绑定函数执行时所在的对象，还会立即执行函数，因此不得不把绑定语句写在一个函数体内。更简洁的写法是采用下面介绍的bind方法。

### 4.3function.prototype.bind()
bind方法用于将函数体内的this绑定到某个对象，然后返回一个新函数。
```
var d = new Date();
d.getTime() // 1481869925657

var print = d.getTime;
print() // Uncaught TypeError: this is not a Date object.
```
上面代码中，我们将d.getTime方法赋给变量print，然后调用print就报错了。这是因为getTime方法内部的this，绑定Date对象的实例，赋给变量print以后，内部的this已经不指向Date对象的实例了。

bind方法可以解决这个问题，让log方法绑定console对象。
```
var print = d.getTime.bind(d);
print() // 1481869925657
```
上面代码中，bind方法将getTime方法内部的this绑定到d对象，这时就可以安全地将这个方法赋值给其他变量了。

下面是一个更清晰的例子。
```
var counter = {
  count: 0,
  inc: function () {
    this.count++;
  }
};

counter.count // 0
counter.inc()
counter.count // 1
```
上面代码中，counter.inc内部的this，默认指向counter对象。如果将这个方法赋值给另一个变量，就会出错。
```
var counter = {
  count: 0,
  inc: function () {
    this.count++;
  }
};

var func = counter.inc;
func();
counter.count // 0
count // NaN
```
上面代码中，函数func是在全局环境中运行的，这时inc内部的this指向顶层对象window，所以counter.count是不会变的，反而创建了一个全局变量count。因为window.count原来等于undefined，进行递增运算后undefined++就等于NaN。

为了解决这个问题，可以使用this方法，将inc内部的this绑定到counter对象。
```
var func = counter.inc.bind(counter);
func();
counter.count // 1
```
上面代码中，bind方法将inc方法绑定到counter以后，再运行func就会得到正确结果。

this绑定到其他对象也是可以的。
```
var obj = {
  count: 100
};
var func = counter.inc.bind(obj);
func();
obj.count // 101
```
上面代码中，bind方法将inc方法内部的this，绑定到obj对象。结果调用func函数以后，递增的就是obj内部的count属性。

bind比call方法和apply方法更进一步的是，除了绑定this以外，还可以绑定原函数的参数。
```
var add = function (x, y) {
  return x * this.m + y * this.n;
}

var obj = {
  m: 2,
  n: 2
};

var newAdd = add.bind(obj, 5);

newAdd(5)
// 20
```
上面代码中，bind方法除了绑定this对象，还将add函数的第一个参数x绑定成5，然后返回一个新函数newAdd，这个函数只要再接受一个参数y就能运行了。

如果bind方法的第一个参数是null或undefined，等于将this绑定到全局对象，函数运行时this指向顶层对象（在浏览器中为window）。
```
function add(x, y) {
  return x + y;
}

var plus5 = add.bind(null, 5);
plus5(10) // 15
```
上面代码中，函数add内部并没有this，使用bind方法的主要目的是绑定参数x，以后每次运行新函数plus5，就只需要提供另一个参数y就够了。而且因为add内部没有this，所以bind的第一个参数是null，不过这里如果是其他对象，也没有影响。

对于那些不支持bind方法的老式浏览器，可以自行定义bind方法。
```
if(!('bind' in Function.prototype)){
  Function.prototype.bind = function(){
    var fn = this;
    var context = arguments[0];
    var args = Array.prototype.slice.call(arguments, 1);
    return function(){
      return fn.apply(context, args);
    }
  }
}
```
bind方法有一些使用注意点。

（1）每一次返回一个新函数

bind方法每运行一次，就返回一个新函数，这会产生一些问题。比如，监听事件的时候，不能写成下面这样。
```
element.addEventListener('click', o.m.bind(o));
```
上面代码中，click事件绑定bind方法生成的一个匿名函数。这样会导致无法取消绑定，所以，下面的代码是无效的。
```
element.removeEventListener('click', o.m.bind(o));
```
正确的方法是写成下面这样：
```
var listener = o.m.bind(o);
element.addEventListener('click', listener);
//  ...
element.removeEventListener('click', listener);
```
（2）结合回调函数使用

回调函数是JavaScript最常用的模式之一，但是一个常见的错误是，将包含this的方法直接当作回调函数。
```
var counter = {
  count: 0,
  inc: function () {
    'use strict';
    this.count++;
  }
};

function callIt(callback) {
  callback();
}

callIt(counter.inc)
// TypeError: Cannot read property 'count' of undefined
```
上面代码中，counter.inc方法被当作回调函数，传入了callIt，调用时其内部的this指向callIt运行时所在的对象，即顶层对象window，所以得不到预想结果。注意，上面的counter.inc方法内部使用了严格模式，在该模式下，this指向顶层对象时会报错，一般模式不会。

解决方法就是使用bind方法，将counter.inc绑定counter。
```
callIt(counter.inc.bind(counter));
counter.count // 1
```
还有一种情况比较隐蔽，就是某些数组方法可以接受一个函数当作参数。这些函数内部的this指向，很可能也会出错。
```
var obj = {
  name: '张三',
  times: [1, 2, 3],
  print: function () {
    this.times.forEach(function (n) {
      console.log(this.name);
    });
  }
};

obj.print()
// 没有任何输出
```
上面代码中，obj.print内部this.times的this是指向obj的，这个没有问题。但是，forEach方法的回调函数内部的this.name却是指向全局对象，导致没有办法取到值。稍微改动一下，就可以看得更清楚。
```
obj.print = function () {
  this.times.forEach(function (n) {
    console.log(this === window);
  });
};

obj.print()
// true
// true
// true
```
解决这个问题，也是通过bind方法绑定this。
```
obj.print = function () {
  this.times.forEach(function (n) {
    console.log(this.name);
  }.bind(this));
};

obj.print()
// 张三
// 张三
// 张三
```
（3）结合call方法使用

利用bind方法，可以改写一些JavaScript原生方法的使用形式，以数组的slice方法为例。
```
[1, 2, 3].slice(0, 1)
// [1]

// 等同于

Array.prototype.slice.call([1, 2, 3], 0, 1)
// [1]
```
上面的代码中，数组的slice方法从[1, 2, 3]里面，按照指定位置和长度切分出另一个数组。这样做的本质是在[1, 2, 3]上面调用Array.prototype.slice方法，因此可以用call方法表达这个过程，得到同样的结果。

call方法实质上是调用Function.prototype.call方法，因此上面的表达式可以用bind方法改写。
```
var slice = Function.prototype.call.bind(Array.prototype.slice);

slice([1, 2, 3], 0, 1) // [1]
```
可以看到，利用bind方法，将[1, 2, 3].slice(0, 1)变成了slice([1, 2, 3], 0, 1)的形式。这种形式的改变还可以用于其他数组方法。
```
var push = Function.prototype.call.bind(Array.prototype.push);
var pop = Function.prototype.call.bind(Array.prototype.pop);

var a = [1 ,2 ,3];
push(a, 4)
a // [1, 2, 3, 4]

pop(a)
a // [1, 2, 3]
```
如果再进一步，将Function.prototype.call方法绑定到Function.prototype.bind对象，就意味着bind的调用形式也可以被改写。
```
function f() {
  console.log(this.v);
}

var o = { v: 123 };

var bind = Function.prototype.call.bind(Function.prototype.bind);

bind(f, o)() // 123
```
上面代码表示，将Function.prototype.call方法绑定Function.prototype.bind以后，bind方法的使用形式从f.bind(o)，变成了bind(f, o)。