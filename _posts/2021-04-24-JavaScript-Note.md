---
title: JavaScript
date: 2021-04-24 15:02
categories: [Note, Language]
tags: [js]     # TAG names should always be lowercase
---

## 数据类型

1. 类型

    - 基本数据类型：String，Boolean，Number，Undefined，Null；
    - 引用数据类型：Object(Array，Date，RegExp，Function)；

2. 类型判定

    ```js
    typeof null // object

    [] instanceof Array // true
    new Date instanceof Date // true

    [].__proto__.constructor // Array()

    ```

    - typeof 返回的是原型链最顶端的结果
    - instanceof 返回的是 A是否是B的实例，即B只需要在A的原型链上即可
    - instanceof 的问题在于，它假定只有一个全局执行环境。如果网页中包含多个框架，那实际上就存在两个以上不同的全局执行环境，从而存在两个以上不同版本的构造函数。如果你从一个框架向另一个框架传入一个数组，那么传入的数组与在第二个框架中原生创建的数组分别具有各自不同的构造函数
    - constructor
    - toString() 是 Object 的原型方法，调用该方法，默认返回当前对象的 [[Class]] 。这是一个内部属性，其格式为 [object Xxx] ，其中 Xxx 就是对象的类型，对于 Object 对象，直接调用 toString()  就能返回 [object Object] 。而其他对象需要通过 call / apply 来调用才能返回正确的类型信息
        > Object.prototype.toString.call('') // [Object Array]

    [参考](https://www.cnblogs.com/onepixel/p/5126046.html)

3. 深浅拷贝

    简单的变量，内存小，我们直接复制不会发生引用；而对于对象这种内存占用比较大的来说，直接让复制的东西等于要复制的，那么就会发生引用

    - 深复制在计算机中开辟了一块内存地址用于存放复制的对象
    - 浅复制仅仅是指向被复制的内存地址，如果原地址中对象被改变了，那么浅复制出来的对象也会相应改变

    深复制的方法

    - Array.from
    - ...

    ```js
    a = [1,2,3]
    b = Array.from(a) // [1,2,3]
    c = [...a] // [1,2,3]
    ```

## 数据劫持

1. Proxy

    ```js
    const p = new Proxy(target, handler)
    ```

    target是劫持的对象，handler则是处理该对象的操作的捕捉器

    handler 对象是一个容纳一批特定属性的占位符对象。它包含有 Proxy 的各个捕获器（trap）.所有的捕捉器是可选的。如果没有定义某个捕捉器，那么就会保留源对象的默认行为。

    - handler.getPrototypeOf() Object.getPrototypeOf 方法的捕捉器。
    - handler.setPrototypeOf() Object.setPrototypeOf 方法的捕捉器。
    - handler.isExtensible() Object.isExtensible 方法的捕捉器。
    - handler.preventExtensions() Object.preventExtensions 方法的捕捉器。
    - handler.getOwnPropertyDescriptor() Object.getOwnPropertyDescriptor 方法的捕捉器。
    - handler.defineProperty() Object.defineProperty 方法的捕捉器。
    - handler.has() in 操作符的捕捉器。
    - handler.get() 属性读取操作的捕捉器。
    - handler.set() 属性设置操作的捕捉器。
    - handler.deleteProperty() delete 操作符的捕捉器。
    - handler.ownKeys() Object.getOwnPropertyNames 方法和 Object.getOwnPropertySymbols 方法的捕捉器。
    - handler.apply() 函数调用操作的捕捉器。
    - handler.construct() new 操作符的捕捉器

    [参考 MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Proxy)

## 闭包

闭包是js语言的一大特性

[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)

里面有一个经典案例

```html
<p id="help">Helpful notes will appear here</p>
<p>E-mail: <input type="text" id="email" name="email"></p>
<p>Name: <input type="text" id="name" name="name"></p>
<p>Age: <input type="text" id="age" name="age"></p>
```

```js
(function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help);
    }
  }
})();
```

无论焦点在哪个input上，显示的都是关于年龄的信息。

原因是赋值给 onfocus 的是闭包。这些闭包是由他们的函数定义和在 setupHelp 作用域中捕获的环境所组成的。这三个闭包在循环中被创建，但他们共享了同一个词法作用域，在这个作用域中存在一个变量item。这是因为变量item使用var进行声明，由于变量提升，所以具有函数作用域。当onfocus的回调执行时，item.help的值被决定。由于循环在事件触发之前早已执行完毕，变量对象item（被三个闭包所共享）已经指向了helpText的最后一项

解决这个问题的一种方案是使用更多的闭包:

```js
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}

function makeHelpCallback(help) {
  return function() {
    showHelp(help);
  };
}

(function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = makeHelpCallback(item.help);
  }
})();
```

另外是使用`匿名闭包`方法，详情仍然是参考该链接

如果不是某些特定任务需要使用闭包，在其它函数中创建函数是不明智的，因为闭包在处理速度和内存消耗方面对脚本性能具有负面影响

## this

参考[阮一峰的网络日志](https://www.ruanyifeng.com/blog/2018/06/javascript-this.html)

常见的例子

```js

var obj = {
  foo: function () { console.log(this.bar) },
  bar: 1
};

var foo = obj.foo;
var bar = 2;

obj.foo() // 1
foo() // 2
```

`this`指的是`函数运行时所在的环境`。对于obj.foo()来说，foo运行在obj环境，所以this指向obj；对于foo()来说，foo运行在全局环境，所以this指向全局环境。这一现象是基于`函数在obj中以其地址存储`

## 运行机制

1. 单线程

    JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题

2. Event Loop

    - 所有同步任务都在主线程上执行，形成一个执行栈（execution context stack）
    - 主线程之外，还存在一个"任务队列"（task queue）。只要异步任务有了运行结果，就在"任务队列"之中放置一个事件
    - 一旦"执行栈"中的所有同步任务执行完毕，系统就会读取"任务队列"，看看里面有哪些事件。那些对应的异步任务，于是结束等待状态，进入执行栈，开始执行

    "任务队列"是一个事件的队列（也可以理解成消息的队列），IO设备完成一项任务，就在"任务队列"中添加一个事件，表示相关的异步任务可以进入"执行栈"了。主线程读取"任务队列"，就是读取里面有哪些事件

    "任务队列"中的事件，除了IO设备的事件以外，还包括一些用户产生的事件（比如鼠标点击、页面滚动等等）。只要指定过回调函数，这些事件发生时就会进入"任务队列"，等待主线程读取

    所谓"回调函数"（callback），就是那些会被主线程挂起来的代码。异步任务必须指定回调函数，当`主线程开始执行异步任务，就是执行对应的回调函数`

    `定时器`

    主线程首先要检查一下执行时间，某些事件只有到了规定的时间，才能返回主线程

    ```setTimeout(fn, 0)```只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。要是当前代码耗时很长，有可能要等很久，所以并没有办法保证，回调函数一定会在setTimeout()指定的时间执行

    `宏任务和微任务`

    |            | 宏任务（macro task）                                                                                                                    | 微任务（micro task）                                                                             |
    | ---------- | --------------------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
    | 发起       | 宿主（Node、浏览器）                                                                                                                    | JS引擎                                                                                           |
    | 具体事件   | script (可以理解为外层同步代码)；setTimeout/setInterval；UI rendering/UI事件；postMessage，MessageChannel；setImmediate，I/O（Node.js） | Promise、Mutaion Observer、Object.observe（已废弃；Proxy 对象替代）、process.nextTick（Node.js） |
    | 运行       | 后运行                                                                                                                                  | 先运行                                                                                           |
    | 触发新Tick | 会                                                                                                                                      | 不会                                                                                             |

3. 内存管理

    | 垃圾回收算法 | 描述                                                                                                                                                                                                       | 缺陷                             |
    | ------------ | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------- |
    | 引用计数法   | "对象是否不再需要”简化定义为“对象有没有其他对象引用到它"                                                                                                                                                   | 循环引用                         |
    | 标记清除法   | 假定设置一个叫做根（root）的对象（在JS里根是全局对象）。垃圾回收器将定期从根开始，找所有从根开始引用的对象，然后找这些对象引用的对象……从根开始，垃圾回收器将找到所有可以获得的对象和收集所有不能获得的对象 | 无法从根对象查询到的对象都被清除 |

    参考[MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Memory_Management)

## 异步

1. 回调函数

    回调函数的优点是简单、容易理解和部署，缺点是不利于代码的阅读和维护，各个部分之间高度耦合（Coupling），流程会很混乱，而且每个任务只能指定一个回调函数

2. 事件监听

    可以绑定多个事件，每个事件可以指定多个回调函数，而且可以"去耦合"（Decoupling），有利于实现模块化。缺点是整个程序都要变成事件驱动型，运行流程会变得很不清晰

3. 发布/订阅

    假定存在一个"信号中心"，某个任务执行完成，就向信号中心"发布"（publish）一个信号，其他任务可以向信号中心"订阅"（subscribe）这个信号，从而知道什么时候自己可以开始执行

    这种方法的性质与"事件监听"类似，但是明显优于后者。因为我们可以通过查看"消息中心"，了解存在多少信号、每个信号有多少订阅者，从而监控程序的运行

4. Promise

   - 创建promise时，它既不是成功也不是失败状态。这个状态叫作`pending`（待定）。
   - 当promise返回时，称为 `resolved`（已解决）
     - 一个成功resolved的promise称为`fullfilled`（实现）。它返回一个值，可以通过将.then()块链接到promise链的末尾来访问该值。 .then()块中的执行程序函数将包含promise的返回值。
     - 一个不成功resolved的promise被称为`rejected`（拒绝）了。它返回一个原因（reason），一条错误消息，说明为什么拒绝promise。可以通过将.catch()块链接到promise链的末尾来访问此原因。

    每一个异步任务返回一个Promise对象，该对象有一个then方法，允许指定回调函数。优点在于，回调函数变成了链式写法，程序的流程可以看得很清楚；如果一个任务已经完成，再添加回调函数，该回调函数会立即执行

    [MDN](https://developer.mozilla.org/zh-CN/docs/Learn/JavaScript/Asynchronous/Promises)

5. async/await

    `async`关键字，保证函数的返回值是一个promise

    ```js
    async function hello() { return "Hello" };
    hello();
    hello.then(console.log('hello'))
    ```

    `await`关键字只在异步函数中起作用。它可以放在任何异步的，基于 promise 的函数之前。它会暂停代码在该行上，直到 promise 完成，然后返回结果值。在暂停的同时，其他正在等待执行的代码就有机会执行了

    Async/await 让你的代码看起来是同步的，在某种程度上，也使得它的行为更加地同步。 await 关键字会阻塞其后的代码，直到promise完成，就像执行同步操作一样。它确实可以允许其他任务在此期间继续运行，但你的代码被阻塞

    经典例题

    ```js
    new Promise((resolve, reject) => {
        console.log("A");
        setTimeout(() => {
            console.log("B");
        }, 0);
        console.log("C");
        resolve();
        console.log("D");
    }).then(() => {
            console.log("E");
            new Promise((resolve, reject) => {
                console.log("F");
                resolve();
                console.log("G");
            }).then(() => {
                    setTimeout(() => {
                        console.log("H");
                    }, 0);
                    console.log("I");
            }).then(() => {
                    console.log("J");
            });
        }).then(() => {
                console.log("K");
        });

    setTimeout(() => {
        console.log("L");
    }, 0);

    new Promise((resolve, reject) => {
        console.log("M");
        resolve();
    }).then(() => {
        setTimeout(() => {
            new Promise((resolve, reject) => {
                console.log("N");
                resolve();
            })
            .then(() => {
                setTimeout(() => {
                    console.log("O");
                }, 0);
            })
            .then(() => {
                console.log("P");
            });
        }, 0);
    });

    console.log("Q");
    ```

    输出

    ```log
    A C D M Q E F G I K J B L N P H O
    ```

    要点：promise是微任务，先于外边的脚本执行，也就是说，其内部的函数与外部的代码可以视为是同步执行的；然后其回调函数也是微任务，在promise执行完后挂起在任务队列前面，或者说加入微任务队列；注意定时器相当于被挂在任务队列的最后面，也即在宏任务队列

    await类型的

    ```js
    async function async1() {
        console.log('async1 start');
        await async2(); // (1)
        console.log('async1 end');
    }
    async function async2() {
        console.log('async2');
    }
    
    console.log('script start');
    
    setTimeout(function() {
        console.log('setTimeout');
    }, 0)
    
    async1();
    
    new Promise(function(resolve) {
        console.log('promise1');
        resolve();
    }).then(function() {
        console.log('promise2');
    });
    console.log('script end');
    ```

    要注意(1)处，由于async2是异步函数，所以它的返回被挂到了微任务队列，所以接下来继续`执行栈`上的任务，即执行promise

    输出如下:

    ```log
    script start
    async1 start
    async2      
    promise1    
    script end
    async1 end
    promise2
    setTimeout
    ```

## ES6

1. 变量声明

    - `const`:声明object的时候其内容可以改变，即只有地址不变；在声明时必须赋值
    - `var`:声明的变量可以[hoisting](https://developer.mozilla.org/zh-CN/docs/Glossary/Hoisting)（变量提升）
    - `let`:和const一样只在最近的花括号内有效

    一些示范

    ```js
    a = 0;
    console.log(a);
    // a = 1
    // var a = 1
    // let a = 1
    // const a = 1

    // 分别尝试上述语句，只有1和2可以正常运行，且输出结果为0
    // var即使可以运行，但是仅可以声明而不能初始化，故为0
    // 由于let和const不能提升，所以其声明会报错（不能到达）
    ```

2. 模板字符串

3. 箭头函数

    - 不需要 function 关键字来创建函数
    - 有且仅有一个表达式时可省略{} 和 return
    - 继承当前上下文的 this 关键字
4. 函数参数默认值可直接在参数列表中赋值

5. 扩展运算符`...`

6. 支持二进制(0b)和八进制(0o)数值表示

7. 对象和数组解构

    ```js
    obj = { a: 1, b: 2, c: 3 }
    const { q, w, e } = obj
    ```

8. 对象超类

    ```js
    var parent = {
        foo() {
            console.log("Hello from the Parent");
        }
    }
    
    var child = {
        foo() {
            super.foo();
            console.log("Hello from the Child");
        }
    }
    
    Object.setPrototypeOf(child, parent);
    child.foo(); 
    // Hello from the Parent
    // Hello from the Child
    ```

9. for ... of | for ... in

10. 类

    ```js
    class Student {
        constructor() {
            console.log("I'm a student.");
        }
    
        study() {
            console.log('study!');
        }
        
        static read() {
            console.log("Reading Now.");
        }
    }
    
    console.log(typeof Student); // function
    let stu = new Student(); // "I'm a student."
    stu.study(); // "study!"
    stu.read(); // "Reading Now."
    ```

    其中继承和超类（注意类的声明不会hoisting）

    ```js
    class Phone {
        constructor() {
            console.log("I'm a phone.");
        }
    }
    
    class MI extends Phone {
        constructor() {
            super();
            console.log("I'm a phone designed by apple");
        }
    }
    ```

## 原型和继承、构造函数

new 的过程

```js
function create(Con,...args){
    //1、创建一个空的对象
    let obj = {}; // let obj = Object.create({});
    //2、将空对象的原型prototype指向构造函数的原型
    Object.setPrototypeOf(obj,Con.prototype); // obj.__proto__ = Con.prototype
    //3、改变构造函数的上下文（this）,并将剩余的参数传入
    let result = Con.apply(obj,args);
    //4、在构造函数有返回值的情况进行判断
    return result instanceof Object?result:obj;
    // 1、如果返回值是基础数据类型，则忽略返回值；
    // 2、如果返回值是引用数据类型，则使用return 的返回，也就是new操作符无效；
}
```
