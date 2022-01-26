## proxy 代理

`proxy`对象的作用是创建某个对象的代理，起到拦截目标对象的默认行为的作用，比如拦截对象的取值，和赋值行为，其基本语法是：

```javascript
var proxy = new Proxy(target, handler);
```

其中 `target`是被代理的原始对象，`handler`是代理配置，在这里可配置拦截行为。  
以最常见的取值、赋值为例：

```javascript
let obj = { name: 'ming' };

let objPro = new Proxy(obj, {
  get(target, propKey) {
    console.log(`get ${propKey}:`);
    return target[propKey];
  },
  set(target, propKey, value) {
    console.log(`set ${propKey}:${value}`);
    target[propKey] = value;
    return true;
  },
});

objPro.name;
objPro.name = 'hong';

/*console log
get name
set name hong 
*/
```

通过上面的代码，`objPro`成了 `obj`的代理，并且重新定义了 get 和 set 方法，当对 `objPro`的属性取值或赋值的时候，就会执行 get 或 set 方法。  
而 `get`和 `set`方法的默认传参有： `target`（被代理对象），`propKey`(当前操作的属性名)，`value`(所赋的值)。所以通过这些参数，就可以实现对被代理对象的操作了。

`handler`的配置除了基础的 get 和 set 外，还有很多其他的配置：

1. `get(target, propKey, receiver)`：拦截对象属性的读取，比如 proxy.foo 和 proxy['foo']。

2. `set(target, propKey, value, receiver)`：拦截对象属性的设置，比如 proxy.foo = v 或 proxy['foo'] = v，返回一个布尔值。

3. `has(target, propKey)`：拦截 propKey in proxy 的操作，返回一个布尔值。

4. `deleteProperty(target, propKey)`：拦截 delete proxy[propKey]的操作，返回一个布尔值。

5. `ownKeys(target)`：拦截 Object.getOwnPropertyNames(proxy)、Object.getOwnPropertySymbols(proxy)、Object.keys(proxy)、for...in 循环，返回一个数组。该方法返回目标对象所有自身的属性的属性名，而 Object.keys()的返回结果仅包括目标对象自身的可遍历属性。

6. `getOwnPropertyDescriptor(target, propKey)`：拦截 Object.getOwnPropertyDescriptor(proxy, propKey)，返回属性的描述对象。

7. `defineProperty(target, propKey, propDesc)`：拦截 Object.defineProperty(proxy, propKey, propDesc）、Object.defineProperties(proxy, propDescs)，返回一个布尔值。

8. `preventExtensions(target)`：拦截 Object.preventExtensions(proxy)，返回一个布尔值。

9. `getPrototypeOf(target)`：拦截 Object.getPrototypeOf(proxy)，返回一个对象。

10. `isExtensible(target)`：拦截 Object.isExtensible(proxy)，返回一个布尔值。

11. `setPrototypeOf(target, proto)`：拦截 Object.setPrototypeOf(proxy, proto)，返回一个布尔值。如果目标对象是函数，那么还有两种额外操作可以拦截。

12. `apply(target, object, args)`：拦截 Proxy 实例作为函数调用的操作，比如 proxy(...args)、proxy.call(object, ...args)、proxy.apply(...)。

13. `construct(target, args)`：拦截 Proxy 实例作为构造函数调用的操作，比如 new proxy(...args)。

具体可参考：

- [阮一峰 ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/proxy)
- [MDN proxy ](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

## Reflect

`Reflect`对象的所有属性和方法都是静态的，不能通过 `new`来对其创建实例（类似 Math 对象）。  
它的作用是以函数的方式，操作某个对象的所有行为，比如对象的取值、赋值行为：

```javascript
let obj = { name: 'ming' };

Reflect.get(obj, 'name'); // 等价于 obj.name
Reflect.set(obj, 'name', 'hong'); // 等价于 obj.name = "hong"
```

如果于 `proxy`相结合使用的话，可以更具优雅的实现对某个对象的代理：

```javascript
let obj = { name: 'ming' };

let objPro = new Proxy(obj, {
  get(target, propKey) {
    console.log(`get ${propKey}:`);
    return Reflect.get(...arguments);
  },
  set(target, propKey, value) {
    console.log(`set ${propKey}:${value}`);
    Reflect.set(...arguments);
    return true;
  },
});
```

`Reflect`除了 get 和 set 外，还提供了很多其他的静态方法：

1. `Reflect.apply(target, thisArgument, argumentsList) `
   对一个函数进行调用操作，同时可以传入一个数组作为调用参数。和 Function.prototype.apply() 功能类似。

2. `Reflect.construct(target, argumentsList[, newTarget]) `
   对构造函数进行 new 操作，相当于执行 new target(...args)。

3. `Reflect.defineProperty(target, propertyKey, attributes)`  
   和 Object.defineProperty() 类似。如果设置成功就会返回 true

4. `Reflect.deleteProperty(target, propertyKey)`  
   作为函数的 delete 操作符，相当于执行 delete target[name]。

5. `Reflect.get(target, propertyKey[, receiver])`  
   获取对象身上某个属性的值，类似于 target[name]。

6. `Reflect.getOwnPropertyDescriptor(target, propertyKey)`  
   类似于 Object.getOwnPropertyDescriptor()。如果对象中存在该属性，则返回对应的属性描述符, 否则返回 undefined.

7. `Reflect.getPrototypeOf(target)`  
   类似于 Object.getPrototypeOf()。

8. `Reflect.has(target, propertyKey)`  
   判断一个对象是否存在某个属性，和 in 运算符 的功能完全相同。

9. `Reflect.isExtensible(target)`  
   类似于 Object.isExtensible().

10. `Reflect.ownKeys(target)`  
    返回一个包含所有自身属性（不包含继承属性）的数组。(类似于 Object.keys(), 但不会受 enumerable 影响).

11. `Reflect.preventExtensions(target)`  
    类似于 Object.preventExtensions()。返回一个 Boolean。

12. `Reflect.set(target, propertyKey, value[, receiver])`  
    将值分配给属性的函数。返回一个 Boolean，如果更新成功，则返回 true。
13. `Reflect.setPrototypeOf(target, prototype)`  
    设置对象原型的函数. 返回一个 Boolean， 如果更新成功，则返回 true。

具体可参考：

- [阮一峰 ECMAScript 6 入门](https://es6.ruanyifeng.com/#docs/reflect)
- [MDN Reflect ](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)
