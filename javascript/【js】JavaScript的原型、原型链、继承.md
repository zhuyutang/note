### 1.原型和原型链的概念

js 在创建一个对象时，比如叫 `obj`，都会给他偷偷的加上一个引用，这个引用指向的是一个对象，比如叫 `yuanxing`，
这个对象可以给引用它的对象提供`属性共享`，比如：`yuanxing`上有个属性 name,可以被 `obj.name`访问到，
这个可以提供属性共享的对象，就称为前面对象的原型

而原型本身也是一个对象，所以它也会有一个自己的原型，这一层一层的延续下去，直到最后指向 null，这就形成的 `原型链`

那 js 的这一种是原型机制是怎么实现的呢？

### 2.原型的实现

我们先从一个例子来看：

```
//code-01
let obj = new Object({name:'xiaomin'})
console.log(obj.name)
console.log(obj.toString())

// xiaomin
// [object Object]
```

我们首先创建了一个对象`obj`，它有一个属性`name`
属性`name`是我们创建的，但是在创建`obj`的时候，并没有给它创建`toString`属性，为什么`obj.toString()`可以访问的到呢？

**prototype 属性**
我们先来看一下`Object.prototype`属性

![](https://img2020.cnblogs.com/blog/2210844/202012/2210844-20201224172510564-1319859521.png)

我们发现这里有`toString`属性，所以其实`Object.prototype`就是`obj`的原型，按照原型的概念，它能够为`obj`提供属性共享
所以当`obj.toString()`的时候，先在`obj`的属性里找，没找到，就在`obj`的原型`Object.prototype`的属性上找，可以找到，所以调用成功

****proto** 属性**
那么`obj`是怎么找到原型的呢？我们打印`obj`属性看一下：

![](https://img2020.cnblogs.com/blog/2210844/202012/2210844-20201224175457642-686612904.png)

我们发现`obj`除了我们创建的`name`以外，还有一个`__proto__`属性，它的值是一个对象，那么它等于什么呢？

![](https://img2020.cnblogs.com/blog/2210844/202012/2210844-20201224175824410-140927632.png)

我们发现`obj.__proto__`指向的就是`Object.prototype`

到此为止，我们可以简单总结一下 js 语言实现原型的基本机制了

- 在创建一个对象`obj`时，会给它加上一个`__proto__`属性
- `__proto__`属性 指向`obj`的`构造函数`的`prototype`对象
- 当访问`obj`的某个属性时，会先在它自己的属性上找，如果没找到，就在它的`原型`(其实就是`__proto__`指向的对象)的属性上找

**构造函数**
这个有一个`构造函数`的概念，其实`构造函数`也就是普通函数，当这个函数被用来 `new`一个新对象时，它就被称为新对象的 `构造函数`
就上面的例子而言，`Objec`就是`obj`的构造函数,
这里要区别一下`Object`和`object`的区别，前者是一个 js 内置的一个函数，后者是 js 的基本数据类型（number,string,function,object,undefined）
![](https://img2020.cnblogs.com/blog/2210844/202012/2210844-20201224181903353-1309506371.png)

### 3.new 实际上做了什么

上面有说到`new`关键字，那么在它实际上做了什么呢？
上面代码`code-01`使用系统内置的函数`Object`来创建对象的，那么我们现在用自己创建的函数来创建一个新对象看看：

```
//code-02

function human(name){
  this.name = name
}
human.prototype.say = function(){
  alert('我叫'+this.name)
}
let xiaomin = new human('xiaoming')

console.log(xiaomin.name)
xiaomin.say()
```

这里的`human`就是新对象`xiaoming`的构造函数
我们把新创建的对象`xiaoming`打印出来看看：
![](https://img2020.cnblogs.com/blog/2210844/202012/2210844-20201225174605381-1657165666.png)
我们看到`xiaoming`有一个属性`name`,并且`xiaoming.__proto__`完全等于构造函数的`human.prototype`,这就是它的`原型`
从这里我们可以总结一下`new`的基本功能：

- 把`构造函数`的 this 指向新创建的对象`xiaoming`
- 为新对象创建`原型`，其实就是把新对象的`__proto__`指向构造函数的`prototype`

**手写 new**
上面我们了解了`new`具体做了什么事情，那么它是怎么实现这些功能的呢？下面我们手写一个函数`myNew`来模拟一下`new`的效果：

```
//code-03
    function human(name, age) {
      this.name = name;
      this.age = age;
    }
    human.prototype.say = function () {
      console.log("my name is " + this.name);
    };

    xiaoming = myNew(human, "xiaoming", 27);

    function myNew() {
      let obj = new Object();
      //取出函数的第一个参数,其实就是 human函数
      let argArr = Array.from(arguments);
      const constructor = argArr.shift();
      // 指定原型
      obj.__proto__ = constructor.prototype;
      //改变函数执行环境
      constructor.apply(obj, argArr);
      return obj;
    }

    xiaoming.say();
```

我们先把新对象`xiaoming`打印出来看一下：
![](https://img2020.cnblogs.com/blog/2210844/202012/2210844-20201225180006315-1452256028.png)
我们发现这和上面的代码`code-02`的效果是一样的
上面代码`code-03`里面如果对`apply`的作用不太熟悉的，可以另外了解一下，其实也很简单，意思就是：在 obj 的环境下，执行 constructor 函数，argArr 是函数执行时的参数，也就是指定了函数的`this`

### 4.继承的实现

其实就上面的内容，就可以对 js 的原型机制有个基本的了解，但是一般面试的时候，如果有问到原型，接下来就会问 能不能实现 `继承`的功能，所以我们来手写一下 原型的继承，其实所用到的知识点都是上面有提到的

**继承的概念**
我们先来说下`继承`的概念：
`继承`其实就是 一个构造函数（子类）可以使用另一个构造函数（父类）的属性和方法，这里有几点注意的：

- 继承是 构造函数 对 另一个构造函数而言
- 需要实现属性的继承，即 `this`的转换
- 需要实现方法的继承，一般就是指 `原型链`的构建

**继承的实现**
基于上面的 3 点要素，我们先直接来看代码:

```
// code-04
   // 父级 函数
    function human(name) {
      this.name = name;
    }
    human.prototype.sayName = function () {
      console.log("我的名字是：", this.name);
    };

    // 子级 函数
    function user(args) {
      this.age = args.age;
      //1.私有属性的继承
      human.call(this, args.name);
      //2.原型的继承
      Object.setPrototypeOf(user.prototype, human.prototype); //原型继承-方法1
      // user.prototype.__proto__ = human.prototype; // 原型继承-方法2
    }
    // 因为重新赋值了prototype，所以放置 user 外部
    // user.prototype = new human();//原型继承-方法3
    // user.prototype = Object.create(human.prototype);//原型继承-方法4

    user.prototype.sayAge = function () {
      console.log("我的年龄是：", this.age);
    };
    let person = new human("人类");
    let xiaoming = new user({ name: "xiaoming", age: 27 });

    console.log("----父类-----");
    console.log(person);
    person.sayName();

    console.log("----子类-----");
    console.log(xiaoming);
    xiaoming.sayName();
    xiaoming.sayAge();

```

我们先来看下打印的结果：
![](https://img2020.cnblogs.com/blog/2210844/202012/2210844-20201229170230390-1028783166.png)
从打印结果，我们可以看到`xiaoming`拥有`person`的属性和方法（`name`,`sayName`）,又有自己私有的属性方法(`age`,`sayAge`),这是因为构造函数`user`实现了对`human`的继承。
其实实现的方法无非也就是我们前面有说到的 作用域的改变和原型链的构造，其中作用域的改变（this 指向的改变）主要是两个方法：call 和 apply，原型链的构造原理只有一个,就是`对象的原型等于其构造函数的prototype属性`，但是实现方法有多种，代码`code-04`中有列出 4 种。
从上面的例子来看原型链的指向是：`xiaoming`->`user.prototype`->`human.prototype`

### 5.class 和 extends

我们可能有看到一些代码直接用 `class` 和 `extends`关键字来实现类和继承，其实这是 ES6 的语法，其实是一种语法糖，本质上的原理也是相同的。我们先来看看基本用法：
**用法**

```
//code-05
   class human {
      //1.必须要有构造函数
      constructor(name) {
        this.name = name;
      }//2.不能有逗号`,`
      sayName() {
        console.log("sayName:", this.name);
      }
    }

    class user extends human {
      constructor(params) {
        //3.子类必须用`super`,调用父类的构造函数
        super(params.name);
        this.age = params.age;
      }
      sayAge() {
        console.log("sayAge:", this.age);
      }
    }

    let person = new human("人类");
    let xiaoming = new user({ name: "xiaoming", age: 27 });

    console.log("----<human> person-----");
    console.log(person);
    person.sayName();

    console.log("----<user> xiaoming-----");
    console.log(xiaoming);
    xiaoming.sayName();
    xiaoming.sayAge();
```

执行结果：
![](https://img2020.cnblogs.com/blog/2210844/202101/2210844-20210105110149968-266644774.png)
我们看到执行的结果和上面的代码`code-04`是一样的，但是代码明显清晰了很多。几个注意的地方：

- class 类中必须要有构造函数`constructor`,
- class 类中的函数不能用 `,`分开
- 如果要继承父类的话，在子类的构造函数中，必须先执行 `super`来调用的父类的构造函数

**相同**
上面有说 class 的写法其实原理上和上面是一样的，我们来验证一下

1. 首先看看`user`和`human`是什么类型
   ![](https://img2020.cnblogs.com/blog/2210844/202101/2210844-20210105114219215-1100952127.png)
   这里看出来了，所以虽然被 class 修饰，本质上还是函数，和代码`code-04`中的`user`,`human`函数是一样的

2. 再来看看 prototype 属性
   ![](https://img2020.cnblogs.com/blog/2210844/202101/2210844-20210105114645551-379449373.png)
   这里看出来`sayName`,`sayAge`都是定义在`human.prototype`和`user.prototype`上，和代码`code-04`中也是一样的

3. 我们再来看看原型链
   ![](https://img2020.cnblogs.com/blog/2210844/202101/2210844-20210105114958863-1894965227.png)
   这与代码`code-04`中的原型链的指向也是一样：`xiaoming`->`user.prototype`->`human.prototype`

**差异**
看完相同点，现在我们来看看不同点：

1. 首先写法上的不同
2. class 声明的函数，必须要用 new 调用
3. class 内部的成员函数没有 prototype 属性，不可以用 new 调用
4. class 内的代码自动是严格模式
5. class 声明不存在变量提升，这一点和 let 一样,比如：

   ```
   //code-06
   console.log(name_var);
   var name_var = "xiaoming";
   //undefined,不会报错，var声明存在变量提升

   console.log(name_let);
   let name_let = "xiaoming";
   // Uncaught ReferenceError: Cannot access 'name_let' before initialization
   //报错，let声明不存在变量提升

   new user();
   class user {}
   // Uncaught ReferenceError: Cannot access 'user' before initialization
   //报错，class声明不存在变量提升

   ```

6. class 内的方法都是不可枚举的，比如:

   ```
     //code-07
     class human_class {
     constructor(name) {
       this.name = name;
     }
     sayName() {
       console.log("sayName:", this.name);
     }
   }
   function human_fun(name) {
     this.name = name;
   }
   human_fun.prototype.sayName = function () {
     console.log("sayName:", this.name);
   };
   console.log("----------human_class-----------");
   console.log("prototype属性", human_class.prototype);
   console.log("prototype 枚举", Object.keys(human_class.prototype));

   console.log("----------human_fun-----------");
   console.log("prototype属性", human_fun.prototype);
   console.log("prototype 枚举", Object.keys(human_fun.prototype));
   ```

   运行结果：
   ![](https://img2020.cnblogs.com/blog/2210844/202101/2210844-20210105130646737-1335190855.png)

### 6.总结

简单总结一下：

- 每个对象在创建的时候，会被赋予一个`__proto__`属性，它指向创建这个对象的构造函数的`prototype`,而`prototype`本身也是对象，所以也有自己的`__proto__`,这就形成了原型链，最终的指向是 `Object.prototype.__proto__ == null`
- 可以通过`new`,`Object.create()`,`Object.setPrototypeOf()`,`直接赋值__proto__`等方法为一个对象指定原型
- `new`操作符实际做的工作是：创建一个对象，把这个对象作为构造函数的 this 环境，并把这个对象的原型（**proto**）指向构造函数的 prototype,最后返回这个对象
- `继承`主要实现的功能是：this 指向的绑定，原型链的构建
- ES6 的语法`class`，`extends`可以提供更为清晰简洁的写法，但是本质上的原理大致相同
