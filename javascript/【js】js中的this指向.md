## 1.this 的使用场景

我们先把`this`的使用场景分为两大类：函数外和函数内：
**函数外的 this**
就是在全局代码里，直接使用 this：

```javascript
'use strict';

let name = 'window';
console.log(this);
console.log(this.name);

// Window
// 'window'
```

从结果看，在函数外的 this 指向很简单，就直接指向的是全局变量`Window`对象，(浏览器环境，以下代码都是在浏览器环境）
而且严格模式或非严格模式都是。

**函数内部的 this**
而在函数内部使用的时候，就可以分为以下几类：

1. 函数独立调用
2. 函数作为对象的方法被调用
3. 函数作为构造函数调用
4. 函数通过 call，apply，bind 调用

**this 指向确定的时间**
在分析不同情况`this`的指向之前，我们确认一个很重要的问题，就是 this 的指向是什么时间确定的。
在说这个问题之前，需要简单说一下`执行上下文`，如果有看过： [js 的闭包、执行上下文、作用域链](https://www.cnblogs.com/zhuyutang/p/14134623.html) 这篇文章，我们就会知道`执行上下文`包含三个部分：

- 变量对象
- 作用域链
- this 指向

我们发现`this`其实`执行上下文`的一部分，当代码需要用到 this 的时候，也是从这里取的。所以`执行上下文`创建的时间，就是 this 确定的时间。
而`执行上下文`的创建时间是：函数被调用，但是还没执行具体代码之前。
所以`this`的指向确定的时间也就明确了：当函数被调用的时候，才开始确定 this 指向。

## 2.分场景分析 this 指向

在了解了`this`被确定的时间后，我们现在来按上面所列出的场景，来具体分析在函数里面的`this`:

**2.1 函数独立调用时**
函数对立调用，其实就是我们最常见的以函数名直接调用函数：

```javascript
// code-01-非严格模式
var name = 'window';
function fun() {
  var name = 'function';
  console.log(this);
  console.log(this.name);
}
fun();
// >> Window
// >> 'window'
```

我们看到，当这样调用函数时，`this`指向的是全局对象`Window`,所以`this.name`就相当于`Window.name`:'window',而不是函数的内部变量 name='function'
这里有一点需要说明的是，这是在`非严格模式`下,那如果是在`严格模式`下呢？我们看下面的例子：

```javascript
// code-01-严格模式

'use strict';
var name = 'window';
function fun() {
  var name = 'function';
  console.log(this);
  console.log(this.name);
}
fun();
// >> undefined
// >> 报错
```

从结果来看，在`严格模式`下，独立调用函数时，函数内部的 this 指向是 `undefined`
其实应该这么说:不管是严格模式还是非严格模式，独立调用函数时，函数内部的 this 指向都是 `undefined`，只不过在非严格模式下，js 会自动把 undefined 的 this 默认指向全局对象：Window

**2.2 函数作为对象的方法调用**
函数作为一个对象的方法调用，我们举例来看：

```javascript
//code-02 作为对象成员方法调用函数
var name = 'window';
var obj = {
  name: 'obj',
  fun: function () {
    console.log(this.name);
  },
  child: {
    name: 'child',
    fun: function () {
      console.log(this.name);
    },
  },
};
// 作为成员方法调用
obj.fun();
// 'obj'

// 多级调用
obj.child.fun();
// 'child'

// 赋值后调用
let fun = obj.fun;
fun();
// 'window'
```

我们下面来分析下上面的代码结果：

- `obj.fun()`
  首先我们从打印的结果来看，这里的 this 等于 obj 对象。
  所以当函数作为某个对象的方法来调用的时候，this 指向这个方法所属的对象。

- `obj.child.fun();`
  从打印的结果来看，这里 this 等于 obj.child 对象。
  所以不管是多少级的调用，this 指向最近的所属对象。

- `var fun = obj.fun; fun();`
  从打印的结果来看，这里 this 等于全局对象 window。window.name = 'window'
  从代码看，这里先做了一个赋值操作，把函数 obj.fun 赋值给了变量 fun, 上面我们有说到 this 的确定时间是在函数被调用的时候，这时候函数并没有被调用，只是做了赋值操作，所以这一步的时候，this 并没有确定。
  当执行到`fun()`的时候，函数被调用，this 在这个时候要确定指向，这时候就相当于是作为独立函数调用，应该指向的是 undefined,但是在非严格模式下，undefined 的 this 会默认指向全局变量 window。
  所以 this.name == window.name == 'window'。如果是严格模式，this.name == undefined.name,会报错。

**2.3 函数作为构造函数调用**
函数作为构造函数的情况，可以分为两种：

1. 构造函数无返回
2. 构造函数有返回值
   a. 返回一个对象
   b. 返回其他非对象的值

下面我们分别来看：

**_构造函数无返回_**
这是构造函数最常用的情况，直接来看代码：

```javascript
//code-03 函数作为构造函数（无返回）

let _this;
function User(name, age) {
  this.name = name;
  this.age = age;
  _this = this;
  console.log(this);
  // {name:"xiaoming",age:27}
}

let xiaoming = new User('xiaoming', 27);
console.log(_this === xiaoming);

// true
```

从结果来看，我们知道当函数作为构造函数的时候，该函数里面的 this 等于这个构造函数 new 的实例对象，就是这里的对象`xiaoming`。从[【机制】JavaScript 的原型、原型链、继承](https://www.cnblogs.com/zhuyutang/p/14145572.html)这篇可以知道操作符 new 实际上做了什么事情。

**_构造函数有返回_**
如果返回的是非对象，则返回值会被忽略，情况等同于无返回。
下面就只讨论返回值为一个对象的情况：

```javascript
//code-03 函数作为构造函数（返回对象）

let _this;
function User(name, age) {
  this.name = name;
  this.age = age;

  _this = this;
  console.log(this);
  // {name:'xiaoming',age:27}

  let obj = {
    name: 'obj',
  };
  return obj;
}
let xiaoming = new User('xiaoming', 27);

console.log(xiaoming);
// {name:'obj'}

console.log(_this === xiaoming);
// false
```

从结果来看，当构造函数返回一个对象时，它 new 出来的实例就等于它返回的对象（xiaoming === obj)，而构造函数的内部 this 并没有起到任何作用。

**2.4 函数通过 call，apply，bind 调用**
call，apply，bind 都是可以指定 this 的值。

```javascript
// code-04 指定this

function fun(name, age) {
  console.log(name, age, this);
}
let obj = {
  name: 'obj',
};

fun.call(obj, 'obj', 27);
fun.apply(obj, ['obj', 27]);
let funBind = fun.bind(obj, 'obj', 27);
funBind();

// 结果返回都一样
// 'obj' 27 {name:obj}
```

call，apply，bind：
相同点：都可以指定函数内部的 this 值，参数的第一个即为 this 的值。
不同点：

- call：fun 参数（name,age）,由 call 函数的第 2,3..参数依次赋值。
- apply：fun 参数（name,age）,由 apply 函数的第 2 个参数赋值，第二个参数是一个数组，所存的值依次赋值给 fun 参数。
- bind：fun 参数（name,age）赋值方式同 call，但 bind 返回的是一个函数，而不是直接执行 fun。

## 3.几种特殊情况

在说明了上面常用情景后，我们来分析几种特殊的情况：

**数组成员**  
当函数作为数组的成员时：

```javascript
// code-05 函数作为数组成员

function arrFun() {
  console.log(this.length);
  console.log(this === arr);
}
let arr = [1, 2, arrFun];
arr[2]();

// 3
// true
```

从结果看，我们知道当函数作为数组的成员的时候，此函数内部的 this 指向的是当前数组。
可以这样理解：arr[2] == arr["2"], 类似于对象的成员方法。

**事件绑定**  
函数作为绑定事件时：

```javascript
// code-06 事件绑定

<button id="btn">点击</button>;

document.getElementById('btn').addEventListener('click', function () {
  console.log(this);
});

// <button id="btn">点击</button>
```

从结果看，我们知道当函数作为事件被绑定时，此函数内部的 this 指向的是绑定了该事件的 dom 元素。

**异步函数：promise，setTimeout**
异步执行函数的时候分为 promise 和 setTimeout 情况（关于异步机制可以参看 [【机制】 JavaScript 的事件循环机制总结 eventLoop](https://www.cnblogs.com/zhuyutang/p/14116167.html)）：

```javascript
// code-07 异步函数

'use strict';

setTimeout(function () {
  console.log('setTimeout:', this);
});

new Promise(function (resolve) {
  console.log('start');
  resolve();
}).then(function () {
  console.log('promise:', this);
});

// start
// promise: undefined
// setTimeout: Window
```

从结果来看，我们知道其实 setTimeout 执行的函数下的 this，相当于是在全局环境下的 this：执行全局变量 Window 对象，严格模式和非严格模式都一样。
promise 下执行的函数其实相当于函数独立执行的情况：严格模式 this 等于 undefined，非严格模式下会默认把 undefined 的 this 指向 Window。

**箭头函数**
其实箭头函数本身没有 this，它里面的 this 指向的是外部作用域中的 this：

```javascript
// code-08 箭头函数

'use strict';

let Obj = {
  name: 'obj',
  fun_1: () => {
    console.log(this);
  },
  fun_2() {
    let fun = () => {
      console.log(this);
    };
    fun();
  },
};
Obj.fun_1();
// Window

Obj.fun_2();
// Obj

function foo() {
  setTimeout(() => {
    console.log(this);
  });
}

foo.call({ id: 42 });
// {id:42}
```

`Obj.fun_1()`
fun_1 是箭头函数，本身没有 this。它的外层作用域就是全局作用域，所以箭头函数的 this 指向的是全局作用域下的 this：Window

`Obj.fun_2()`
fun_2 函数内部的 fun 是箭头函数，本身没有 this。它的外层作用域就是 fun_2，而 fun_2 的 this 是调用它的对象 Obj，所以箭头函数的 this 指向的也是 Obj。

`foo.call({ id: 42 })`
foo 函数用 call 调用，于是 foo 的 this 为{id：42}。本来 setTimeout 内部的函数 this 指向的是 Widow，但是因为它是箭头本身没有 this，箭头函数的 this 指向的是外部作用域的 this，在这里就是 foo 的 this：{id：42}。

-- 完 --
