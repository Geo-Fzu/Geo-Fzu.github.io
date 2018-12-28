---
title: ES6 Promise 对象
date: 2016-07-27 22:26:55
tags: [ES6, JavaScript, 异步]
categories: tech
---
`Promise` 是异步编程的一种解决方案，比传统的回调函数更合理和强大。Promise 保存未来才会结束的事件（通常是一个异步操作）的结果，从 Promise 中可以获得异步操作的消息。
<!--more-->
# 1. Promise 的三种状态

+ Promise 对象代表一个异步操作，有三种状态：
    * `Pending` （进行中）
    * `Resolved` （已完成， Fulfilled）
    * `Rejected` （已失败）
只有异步操作的结果可以决定当前是哪一种状态，任何其他操作不能改变这个状态。
+ Promise 只有两种状态改变的可能：
    * 从 Pending 变为 Resolved
    * 从 Pending 变为 Rejected
**一旦状态改变，就不会再改变。**

有了 Promise 对象，可以将异步操作以同步操作的流程表达出来，避免了层层嵌套的回调函数（callback hell）。

Promise 一旦新建就会立即执行，无法中途取消。如果不设置回调函数，Promise 内部抛出的错误不会反应到外部。

# 2. Promise 的基本用法

## 2.1 Promise 的构造和使用
+ Promise 的构造
Promise 是一个构造函数，用来生成 Promise 实例。下面代码构造了一个 Promise 实例：
``` javascript
var promise = new Promise(function(resolve, reject) {
    // Promise 构造函数接收一个函数作为参数，该函数的两个参数分别是 resolve 和 reject，这是两个函数。

    // ... 异步操作代码

    if(/*异步操作成功*/){
        resolve(value);
    } else {
        reject(error);
    }
})
```
    * `resovle` 将 Promise 对象状态从 Pending 变为 Resolved，在异步操作成功时调用。并且将异步操作的结果，作为参数传递出去。
    * `reject` 将 Promise 对象状态从 Pending 变为 Rejected，在异步操作失败时调用。将异步操作报出的错误，作为参数传递出去。
+ Promise 的使用
Promise 实例生成之后，可以用 `then` 方法分别指定 Resolved 和 Rejected 状态的回调函数：
``` javascript
promise.then(function(value) {
    // promise 中的异步操作成功时执行，即 Promise 状态变为 Resolved，value 为 resolve 返回的值
    }，function(err) {
    // 异步操作失败时执行的回调函数，即 Promise 状态变为 Rejected。
    // 这个函数是可选的。
})
```

## 2.2 Promise 的示例
+ 简单例子：
``` javascript
function timeout(ms) {
  return new Promise((resolve, reject) => {
    setTimeout(resolve, ms, 'done');
    // setTimeout(callback, delay[, ...arg])
    // [, ...arg] Optional arguments to pass when the callback is called.
  });
}

timeout(100).then((value) => {
  console.log(value);
  // 'done'
});
```
+ Promise 创建后立刻执行：
``` javascript
let promise = new Promise(function(resolve, reject) {
  console.log('Promise');
  resolve();
});

promise.then(function() {
  console.log('then');
});

console.log('Hi!');

// Promise
// Hi!
// then
```
上面代码中，Promise 新建后立即执行，所以首先输出 “Promise”。`then` 方法指定的回调函数，将在当前脚本所有同步任务完成后才会执行，所以最后输出 “then”。
+ resolve 函数传递另一个 Promise 实例
如果在调用 resolve 和 reject 函数时带有参数，那么它们的参数会被传递给回调函数。
`reject` 通常接收 Error 对象的实例作为参数，表示抛出的错误；`resolve` 函数的参数可以是正常的值，还可能是另一个 Promise 对象实例。
这表示异步操作的结果可能是一个值，也可能是另一个异步操作：
``` javascript
var p1 = new Promise(function(resolve, reject) {
    // ...
    })

var p2 = new Promise(function(resolve, reject) {
    // ...

    resolve(p1);
    // p2 的 resolve 方法将 p1 作为参数，即一个异步操作的结果是返回另一个异步操作。
    })
```
这时， p1 的状态就会传递给 p2，也就是说，p1 决定了 p2 的状态。
如果 p1 状态是 Pending，那么 p2 的回调函数就会等待 p1 的状态改变；如果 p1 的状态已经是 Resolved 或者 Rejected，那么 p2 的回调函数将会立即执行:
``` javascript
var p1 = new Promise(function(resolve, reject) {
    setTimeout(() => {
        console.log('promise in p1')
        reject(new Error('fail'))
    }, 3000)
})

var p2 = new Promise(function(resolve, reject) {
    setTimeout(() => {
        console.log('promise in p2')
        resolve(p1)
    }, 1000)
})

p2
    .then(result => console.log(result))
    .catch(error => console.log(error))

    // 输出结果：
    // promise in p2
    // promise in p1
    // [Error: fail]
```

# 3. Promise 的方法

## 3.1 Promise.prototype.then()
Promise 实例具有 then 方法，定义在 Promise.prototype 上。它的作用是为 Promise 实例添加状态改变时的回调函数。
then 方法的第一个参数是 `Resolved` 状态的回调函数，第二个参数（可选）是 `Rejected` 状态的回调函数。
**then 方法返回的是一个新的 Promise 实例（注意，不是原来那个 Promise 实例）。因此可以采用链式写法，即 then 方法后面再调用另一个 then 方法。**
采用链式的 then，可以指定一组按照次序调用的回调函数。这时，前一个回调函数，有可能返回的还是一个 Promise 对象（即有异步操作），这时后一个回调函数，就会等待该 Promise 对象的状态发生变化，才会被调用：
``` javascript
var promise = new Promise(function(resolve, reject) {
    setTimeout(resolve, 1000, 'promise chain')
})

// 如果 then 返回的不是 Promise 对象
promise.then((data) => {
    setTimeout(() => {
        console.log("1", data)
    }, 1000)
    return data
}).then((data) => {
    console.log("2", data)
})
// 由于第一个 then 函数里执行的是异步操作，所以先执行第二个 then，执行结果为：
// 2 promise chain
// 1 promise chain

// 如果返回的是一个 Promise 对象：
promise.then((data) => {
    return new Promise(function(resolve, reject) {
            setTimeout(() => {
                console.log("1", data)
                resolve(data)
            }, 1000)
        })
}).then((data) => {
    console.log("2", data)
})
// 返回的是一个 Promise 对象，所以第二个 then 函数会等第一个 then 函数中的异步操作执行完成后，再根据其状态执行回调函数，执行结果为：
// 1 promise chain
// 2 promise chain
```

## 3.2 Promise.prototype.catch()
Promise.protype.catch 与 .then(null, rejectFun) 是相同的作用，用于指定发生错误时的回调函数。
``` javascript
promise.then(function() {
    // ...
    }).catch(function() {
    // 处理 promise 和前一个回调函数运行时发生的错误
    })
```
下面是一个例子：
``` javascript
var p1 = new Promise(function(resolve, reject) {
    setTimeout(() => {
        console.log('promise in p1')
        reject(new Error('fail'))
    }, 3000)
})

var p2 = new Promise(function(resolve, reject) {
    setTimeout(() => {
        console.log('promise in p2')
        resolve(p1)
    }, 1000)
})

p2
    .then(result => console.log(result))
    .catch(error => console.log(error))

    // promise in p2
    // promise in p1
    // [Error: fail]
```
**注：**如果 Promise 状态已经变成 Resolved 那么再抛出错误是无效的。
Promise 对象的错误具有*冒泡*性质，会一直向后传递，直到被捕获为止。也就是说，错误总是会被下一个 catch 语句捕获。
**catch 方法返回的还是一个 Promise 对象，因此后面还可以接着调用 then 方法。**

## 3.3 Promise.all()
Promise.all方法用于将多个Promise实例，包装成一个新的Promise实例。
``` javascript
var p = Promise.all([p1, p2, p3])
```
+ p的状态由 p1、 p2、 p3决定，分成两种情况。
    * 只有 p1、 p2、 p3的状态都变成 Resovled， p 的状态才会变成 Resovled，此时 p1、 p2、 p3的返回值组成一个数组，传递给 p 的回调函数。
    * 只要 p1、 p2、 p3之中有一个被 Rejected，p 的状态就变成 Rejected，此时第一个被 reject 的实例的返回值，会传递给 p 的回调函数。

## 3.4 Promise.race()
Promise.race 方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。
``` javascript
var p = Promise.race([p1,p2,p3]);
```
只要 p1、 p2、 p3 之中有一个实例率先改变状态， p 的状态就跟着改变。那个率先改变的 Promise 实例的返回值，就传递给 p 的回调函数。

## 3.5 Promise.resolve()
有时需要将现有对象转为 Promise 对象，Promise.resolve 方法就起到这个作用。
+ 如果参数是 Promise 实例，那么 Promise.resolve 将不做任何修改、原封不动返回这个实例。
+ 如果参数是 `thenable` 对象，例如：
``` javascript
let thenable = {
    then: function(resolve, reject) {
        resolve('thenable')
    }
}

// Promise.resovle 方法会将这个对象转为 Promise 对象，然后就立即执行 thenable 对象的 then 方法。

let promise = Promise.resolve(thenable)
promise.then(function(value){
    console.log(value)
    })
```
上面代码中，thenable 对象的 then 方法执行后，对象 promise 的状态就变为 Resolved，从而立即执行最后那个 then 方法指定的回调函数，输出 'thenable'。
+ 如果参数是一个原始值，或者是一个不具有 then 方法的对象，则 Promise.resolve 方法返回一个新的 Promise 对象，状态为 Resolved。
+ Promise.resolve 方法允许调用时不带参数，直接返回一个 Resolved 状态的 Promise 对象。

## 3.6 Promise.reject()
Promise.reject(reason) 方法也会返回一个新的 Promise 实例，该实例的状态为 Rejected。它的参数用法与 Promise.resolve 方法完全一致。

> [参考链接](http://es6.ruanyifeng.com/#docs/promise "ECMAScript 6 入门")