---
title: ES6 Generator
date: 2016-08-05 14:39:43
tags: [ES6, JavaScript, 异步]
categories: tech
---
# 一、 基本概念

`Generator` 函数是 ES6 提供的一种异步编程解决方案，语法行为与传统函数完全不同。它的作用是在执行一个函数的时候，可以在某个点暂停执行，做一些其它工作，然后再返回这个函数继续执行，甚至带一些新的值。
执行 Generator 函数会返回一个迭代器对象 （Iterator），说明它是一个迭代器对象生成函数。返回的迭代器可以依次遍历 Generator 函数内部的每一个状态。
形式上，Generator 函数是一个普通函数，但是有两个特征：
1. `function` 关键字与函数名之间有一个 `*` 号；
2. 函数内部使用 `yield` 语句。yield 是一个关键字，和 return 类似，不同点在于 yield 语句可在一个函数中出现多次。*yield 表达式暂停了 Generator 函数的执行，然后可以从暂停的地方恢复执行*。
    + yield 后面的表达式作为迭代器 next 函数的返回值；
    + 迭代器 next 函数的参数作为上一次 yield 的返回值。

Generator 函数的调用方法与普通函数一样，也是在函数名后面加上一对圆括号。*不同的是，调用 Generator 后，该函数并不执行，返回的也不是函数运行结果，而是一个迭代器对象。*
<!--more-->

# 二、 与 Iterator 的关系

在 Iterator 那一篇文章中我们有说过，调用对象的 Symbol.iterator 方法，会返回该对象的一个迭代器对象。由于 Generator 就是一个迭代器生成函数，所以可以把它复制给 对象的 Symbol.iterator 属性，部署该对象的 Iterator 接口:
``` javascript
var Iterable = {}

Iterable[Symbol.iterator] = function*() {
    yield 1
    yield 2
    yield 3
}

console.log(...Iterable)

// 1 2 3
// [Finished in 2.8s]
```

之前介绍过，for...of 循环、扩展运算符、结构赋值等都是调用的迭代器接口。所以我们可以直接将 Generator 返回的对象作为参数，传入这些方法中：
``` javascript
function* gen() {
    yield 1
    yield 2
    yield 3
}

console.log(...gen())

// 1 2 3
// [Finished in 2.7s]
```

# 三、 Generator.prototype.return() 方法

`return` 方法可以返回给定的值，并且中介遍历 Generator 函数。如果 return 不提供参数，则返回值的 value 属性为 undefined。
``` javascript
function* gen() {
    yield 1
    yield 2
    yield 3
}

var g = gen()

console.log(g.next())
console.log(g.return('complete'))
console.log(g.next())

// { value: 1, done: false }
// { value: 'complete', done: true }
// { value: undefined, done: true }
// [Finished in 2.7s]
```

# 四、 yield* 语句
yield* 用来遍历一个迭代器对象：
``` javascript
function* inner() {
    yield 'hello!';
}

function* outer1() {
    yield 'open';
    yield inner();
    yield 'close';
}

var gen = outer1()
console.log(gen.next().value)
console.log(gen.next().value)
console.log(gen.next().value)

function* outer2() {
    yield 'open'
    yield * inner()
    yield 'close'
}

var gen = outer2()
console.log(gen.next().value)
console.log(gen.next().value)
console.log(gen.next().value)

// open
// GeneratorFunctionPrototype { _invoke: [Function: invoke] }
// close

// open
// hello!
// close
// [Finished in 2.7s]
```

# 五、 异步操作的同步化表达 

Generator 函数的暂停执行的效果，意味着可以把异步操作写在 yield 语句里面，等到调用 next 方法时再往后执行。这实际上等同于不需要写回调函数了，因为异步操作的后续操作可以放在 yield 语句下面，反正要等到调用 next 方法时再执行。所以，Generator 函数的一个重要实际意义就是用来处理异步操作，改写回调函数。

Ajax是典型的异步操作，通过Generator函数部署Ajax操作，可以用同步的方式表达。
``` javascript
function* main() {
  var result = yield request("http://some.url");
  var resp = JSON.parse(result);
    console.log(resp.value);
}

function request(url) {
  makeAjaxCall(url, function(response){
    it.next(response);
  });
}

var it = main();
it.next();
```
上面代码的main函数，就是通过 Ajax 操作获取数据。可以看到，除了多了一个 yield，它几乎与同步操作的写法完全一样。注意，makeAjaxCall 函数中的 next 方法，必须加上 response 参数，因为 yield 语句构成的表达式，本身是没有值的，总是等于 undefined。