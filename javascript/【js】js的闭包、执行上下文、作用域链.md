### 1.从闭包说起

**什么是闭包**

> 一个函数和对其周围状态（词法环境）的引用捆绑在一起，这样的组合就是闭包。
> 也就是说，闭包让你可以在一个内层函数中访问到其外层函数的作用域。
> 在 JavaScript 中，每当创建一个函数，闭包就会在函数创建的同时被创建出来。
>
> 上面是 MDN 对`闭包`的解释，这几句话可能不太好懂，没关系，我们先来看下能懂的：

- `闭包`是和函数有关
- 这个函数可以访问它外层函数的作用域
- 从定义看，每个函数都可以称为`闭包`

虽然从定义来看，所有函数都可以称为`闭包`，但是当我们在讨论它的时候，一般是指这种情况：

```
    //code-01
    function cat() {
      var name = "小猫";
      function say() {
        console.log(`my name is ${name}`);
      }
      return say;
    }
    var fun = cat();
    //---cat函数已经执行完,下面却还能够访问到 say函数的内部变量 name
    fun();
    //> my name is 小猫
```

当一个函数的返回值是一个内部函数时（cat 函数返回 say 函数），在这个函数已经执行完毕后，这个返回的内部函数还可以访问到已经执行完毕的函数的内部变量，就像 `code-01`中 fun 可以访问到 cat 函数的 name，一般我们谈论的`闭包`就是指这种情况。
那么这是什么原因呢？这就涉及到函数的`作用域链`和`执行上下文`的概念了,我们下面分别来说。

###2.执行上下文
**定义**
什么是执行上下（Execution context ）呢？简单来说就是全局代码或函数代码执行的时候的环境，它包含三个部分内容：

- 1.变量对象(Variable object,vo)，
- 2.作用域链(Scope chain,sc)
- 3.this 的指向(这篇先不谈)

我们用一个对象来表示：

```
  EC = {
      vo：{},
      sc：[],
      this
  }
```

然后代码或函数需要什么变量的时候，就会在这里面找。

**创建时间**
执行上下文（EC）是什么时候创建的呢？这里分为两种情况：

- 全局代码：代码开始执行，但是还没有执行具体代码之前
- 函数代码：函数要执行的时候，但是还没值执行具体代码之前

其实如果把全局的代码理解为一个大的函数，这两者就可以统一了。
每一个函数都会创建自己的`执行上下文`，他们以栈的形式存储在一起，当函数执行完毕，则把它自己的`执行上下文`出栈，这就叫`执行上下文栈`(Execution context stack,ECS)
下面我们通过一段代码实例来看一下

**声明语句与变量提升**
具体分析之前，我们先来说`声明语句`，什么是`声明语句`呢？

- `声明语句`是用来声明一个变量，函数，类的语句
- 比如：var，let，const，function,class
- 其中 var 和 function 会造成`变量提升`，其他不会，如果 var 和 function 同名的话，则函数声明优先
  那什么是变量提升呢？

```
    // code-02
    console.log(varVal); // 输出undefined
    console.log(fun); // 输出  fun(){console.log('我是函数体') }，
    //console.log(letVal) //报错 letVal is not defined

    var varVal = "var 声明的变量";
    let letVal = "let 声明的变量";

    function fun() {
      console.log("我是函数体");
    }
    var fun = "function"; //与函数同名，函数优先，但是可以重新赋值

    console.log(varVal); // >> "var 声明的变量"
    console.log(letVal); // >> "let 声明的变量"
    //fun(); // 报错,因为fun被赋值为'function'字符串了

    var name = "xiaoming";
```

在 js 执行代码的时候，会先扫一遍代码，把 var，function 的声明先执行，var 声明的变量会先赋值为 undefined，function 声明的函数会直接就是函数体,这就叫`变量提升`,而其他的声明，比如 let，则不会。
所以在变量赋值之前，`console.log(varVal)`,`console.log(fun)`可以执行，而`console.log(letVal)`则会报错。
其中 fun 被重新声明为'function'字符串，但是在变量提升的时候，函数优先，所以`console.log(fun)`打印出来的是函数体，而代码执行赋值语句的时候，fun 被赋值成了字符串，所以`fun()`会报错

**代码执行过程分析--变量对象**
我们先上一段简单的代码，通过这段代码，来分析一下 `执行上下文`创建和作用的过程，对其内容我们先只涉及`变量对象`vo:

```
//code-03
var name = 'xiaoming'

function user(name){
   var age = 27
   console.log(`我叫${name},今年${age}`)
}
user(name)
console.log(name)
```

我们现在来分析一下这段代码执行过程中，执行上下文的作用过程，会加入`变量对象`vo，`作用域链`scope 会在下面讲，this 的指向这次不讲，所以就不加上去了

1.代码执行之前，先创建 全局的执行上下文 G_EC,并压入执行上下栈 ECS

```
ECS = [
  G_EC : {
    vo:{
      name:undefined,
      user(name){
        var age = 27
        console.log(`我叫${name},今年${age}`)
      },
    },
    sc
  }
]
```

2.代码开始执行，name 被赋值，执行 user(name) 3.函数执行的时候，具体代码还没执行之前，创建`函数执行上下文`user_EC，并压入 ECS

```
ECS = [
  user_EC : {
    vo:{
      name:undefined,
      age:undefined,
    },
    sc
  },
  G_EC : {
    vo:{
      name:'xiaoming',
      user(name){
        var age = 27
        console.log(`我叫${name},今年${age}`)
      }
    },
    sc
  }
]
```

4.开始执行函数代码，给形参 name 赋值，变量 age 赋值，执行 console.log 的时候需要变量`name`，`age`,于是从它自己的`执行上下文`user_EC 中的`变量对象`vo 里开始查找

```
ECS = [
  user_EC : {
    vo:{
      name:'xiaoming',
      age:27,
    },
    sc
  },
  G_EC : {
    vo:{
      name:'xiaoming',
      user(name){
        var age = 27
        console.log(`我叫${name},今年${age}`)
      }
    },
    sc
  }
]
```

5.发现找到了，于是打印 `我叫xiaoming,今年27`,至此函数 user 执行完毕了，于是把其对应的`执行上下文`user_EC 出栈

```
ECS = [
  G_EC : {
    vo:{
      name:'xiaoming',
      user(name){
        var age = 27
        console.log(`我叫${name},今年${age}`)
      }
    },
    sc
  }
]
```

6.代码继续执行，console.log(name),发现需要变量那么，于是从它自己的`执行上下文`中的`变量对象`开始查找，也就是 G_EC 中的 vo，顺利找到，于是打印"xiaoming" 7.至此代码执行结束，但全局的执行上下文好像要等到当前页面关闭才出栈（浏览器环境）

###3.作用域链
上面我们分析代码执行过程的时候，有说到如果要用到变量的时候，就从当前`执行上下文`中的`变量对象`vo 里查找，我们刚好是都有找到。
那么如果当前`执行上下文`中的`变量对象`中没有需要用的变量呢？根据我们的经验，它会从父级的作用域来查找，那么这是根据什么来查找的呢?
所有接下来我们继续来看 '作用域链'（scope chain,sc），它也是`执行上下文`得另一个组成部分。
** 函数作用域 **
在说`执行上下`中的`作用域链`之前，我们要先来看看`函数作用域`，那么这是个什么东西呢？

- 每一个函数都有一个内部属性【scope】
- 它是函数创建的时候构建的
- 它是一个列表，会把函数的所有父辈的`执行上下`中的`变量对象`存在其中
  举个例子：

```
//code-04
function fun_1(){
  function fun_2(){}
}
```

1.我们看上面的代码，当 fun_1 函数创建的时候，它的父级`执行上下文`是全局执行上下文 `G_EC`,所以 fun_1 的`函数作用域`【scope】为：

```
fun_1.scope = [
  G_EC.vo
]
```

2.当 fun_2 函数创建的时候，它的所有父级`执行上下文`有两个，一个是全局执行上下文 `G_EC`, 还有一个是函数 fun_1 的执行上下文 `fun_1_EC`, 所以 fun_2 的`函数作用域`【scope】为：

```
fun_1.scope = [
  fun_1_EC.vo,
  G_EC.vo
]
```

**执行上下文的作用域链**
上面我们说的是`函数作用域`,它包含了所有父级执行上下的变量对象，但是我们发现它没有包含函数自己的变量对象，因为这个时候函数只是声明了，还没有执行，而函数的`执行上下文`是在函数执行的时候创建的。
当函数执行的时候，会创建函数的`执行上下文`,从上面我们知道，这个时候会创建`执行上下文`的`变量对象`vo，而赋值`执行上下文`的`作用域链`sc 的时候，会把 vo 加在 scope 前面，作为一个队列，赋值给`作用域链`，
就是说：`EC.sc = [EC.vo,...fun.scope]`,我们下面举例说明，这段代码与 code-03 的区别只是不给函数传参，所以会用到父级作用域的变量。

```
//code-05
var name = 'xiaoming'

function user(){
   var age = 27
   console.log(`我叫${name},今年${age}`)
}
user()
console.log(name)
```

1.代码执行之前，先创建 全局的执行上下文 G_EC,并压入执行上下栈 ECS,同时赋值`变量对象`vo、`作用域链`sc，注意：当函数 user 被声明的时候，会带有`函数作用域`user.scope

```
ECS = [
  G_EC : {
    vo:{
      name:undefined,
      user // user.scope:[G_EC.vo]
    },
    sc:[G_EC.vo]
  }
]
```

2.代码开始执行，name 被赋值，执行 user() 3.函数执行的时候，具体代码还没执行之前，创建`函数执行上下文`user_EC，并压入 ECS,同时赋值`变量对象`vo 和`作用域链`sc:

```
ECS = [
  user_EC : {
    vo:{
      age:undefined,
    }，
    sc:[user_EC.vo, ...user.scope]
  },
  G_EC : {
    vo:{
      name:'xiaoming',
      user // user.scope:[G_EC.vo]
    },
    sc:[G_EC.vo]
  }
]
```

4.开始执行函数代码，给变量 age 赋值，执行 console.log 的时候需要变量`name`，`age`,这里我们上面说是从`变量对象`里找，这里更正一下，其实是从`作用域链`中查找

```
ECS = [
  user_EC : {
    vo:{
      age:27,
    },
    sc:[user_EC.vo, ...user.scope]
  },
  G_EC : {
    vo:{
      name:'xiaoming',
      user, // user.scope:[G_EC.vo]
    },
    sc:[G_EC.vo]
  }
]
```

5.我们发现在`作用域链`的第一个对象中(user_EC.vo)找到了 age,但是没有`name`，于是开始查找`作用域链`的第二个对象，依次往下找，如果都没找到，则会报错。
这里的话，我们发现`作用域链`的第二个元素 user.scope 析构出来的，也就是 G_EC.vo,这个里面有找到 name='xiaoming'
于是打印 `我叫xiaoming,今年27`,至此函数 user 执行完毕了，于是把其对应的`执行上下文`user_EC 出栈

```
ECS = [
  G_EC : {
    vo:{
      name:'xiaoming',
      user, // user.scope:[G_EC.vo]
    },
    sc:[G_EC.vo]
  }
]
```

6.代码继续执行，console.log(name),发现需要变量那么，于是从它自己的`执行上下文`中的`作用域链`开始查找，在第一个元素 G_EC.vo 就顺利找到，于是打印"xiaoming" 7.至此代码执行结束，

###4.回归到闭包的问题
到此为止我们介绍完了`执行上下`文，那么现在我们回归到刚开始的`闭包`为什么能访问到已经执行完毕了的函数的内部变量问题。我们再来回顾一下代码：

```
    //code-06
    function cat() {
      var name = "小猫";
      function say() {
        console.log(`my name is ${name}`);
      }
      return say;
    }
    var fun = cat();
    fun();
```

我们来照上面的步骤来分析下代码： 1.代码执行之前，先创建 全局的执行上下文 G_EC,并压入执行上下栈 ECS,同时赋值`变量对象`vo、`作用域链`sc

```
ECS = [
  G_EC : {
    vo:{
      fun:undefined,
      cat, // cat.scope:[G_EC.vo]
    },
    sc:[G_EC.vo]
  }
]
```

2.代码开始执行，执行 cat()函数 3.函数执行的时候，具体代码还没执行之前，创建`函数执行上下文`cat_EC，并压入 ECS,同时赋值`变量对象`vo 和`作用域链`sc:

```
ECS = [
  cat_EC : {
    vo:{
      name:undefined,
      say, // say.scope:[cat_EC.vo,G_EC.vo]
    }，
    sc:[cat_EC.vo, ...cat.scope]
  },
  G_EC : {
    vo:{
      fun:undefined,
      cat, // cat.scope:[G_EC.vo]
    },
    sc:[G_EC.vo]
  }
]
```

4.开始执行函数代码，给变量 name 赋值，然后返回 say 函数，这个时候函数执行完毕，它的值被付给变量 fun,它的`执行上下文`出栈

```
ECS = [
  G_EC : {
    vo:{
      fun:say, // say.scope:[cat_EC.vo,G_EC.vo]
      cat // cat.scope:[G_EC.vo]
    },
    sc:[G_EC.vo]
  }
]
```

5.代码继续执行，到了 fun(), 6.当函数要执行，还没执行具体代码之前，创建`函数执行上下文`fun_EC，并压入 ECS,同时赋值`变量对象`vo 和`作用域链`sc:

```
ECS = [
  fun_EC : {
    vo:{}，
    sc:[fun_EC.vo, ...fun.scope]//fun==cat,所以fun.scope = say.scope = [cat_EC.vo,G_EC.vo]
  },
  G_EC : {
    vo:{
      fun:say, // say.scope:[cat_EC.vo,G_EC.vo]
      cat // cat.scope:[G_EC.vo]
    },
    sc:[G_EC.vo]
  }
]
```

7.函数 fun 开始执行具体代码：`console.log(`my name is ${name}`)`,发现需要变量`name`,于是从他的 fun_EC.sc 中开始查找，第一个 fun_EC.vo 没有，于是找第二个 cat_EC.vo,发现这里有 name="小猫",
于是打印 `my name is 小猫`,至此函数 fun 执行完毕了，于是把其对应的`执行上下文`fun_EC 出栈

```
ECS = [
  G_EC : {
    vo:{
      fun:say, // say.scope:[cat_EC.vo,G_EC.vo]
      cat // cat.scope:[G_EC.vo]
    },
    sc:[G_EC.vo]
  }
]
```

8.至此代码执行结束

到这里我们知道`闭包`为什么可以访问到已经执行完毕的函数的内部变量，是因为在的`执行上下文`中的`作用域链`中保存了变量的引用，而保存的引用的变量不会被垃圾回收机制所销毁。

**闭包的优缺点**
优点：

1. 可以创建拥有私有变量的函数，使函数具有封装性
2. 避免全局变量污染

缺点：

1. 增大内存消耗

**参考** 1.[JavaScript 深入之词法作用域和动态作用域](https://github.com/mqyqingfeng/Blog/issues/3) 2.[JavaScript 深入之执行上下文栈](https://github.com/mqyqingfeng/Blog/issues/4) 3.[setTimeout 和 setImmediate 到底谁先执行，本文让你彻底理解 Event Loop](https://www.cnblogs.com/dennisj/p/12550996.html)
