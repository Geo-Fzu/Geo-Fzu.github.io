---
title: NodeJs 测试用例 mocha、 should 以及 supertest
date: 2016-07-30 23:15:20
tags: [NodeJs, 测试]
categories: tech
---

# 一、为什么要使用测试

首先，单元测试是衡量代码质量的一个重要标准，Github 上受欢迎的项目大多是有 test 文件夹，加上单元测试能让你的 module 值得别人信赖。
其次，个人认为作为一个负责任的程序猿，每当你写完一个功能模块，你至少也是要手动去测试。而使用代码进行单元测试比起手动的来更容易复用，通过简单的重用实现更多的功能。并且在代码的改动之后，单元测试只需针对改动的部分进行测试，而不是每次都从头开始，提高了效率。

<!--more-->

---

# 二、测试框架 - Mocha

Mocha 是一个功能丰富的 JavaScript 测试框架，它能运行在 Node.js 和浏览器中，支持 `BDD` （Behavior Driven Development）、 `TDD` （Test Driven Development） 等形式的测试。

> 要了解更多可以访问 Mocha 的官网 [mochajs.org](http://mochajs.org/).

让我们从一个最简单的例子开始：

```javascript
// unit.test.js
describe("mocha test describe", () => {
  it("mocha test it", () => {});
});
```

执行 `mocha unit.test.js`，结果如下：

```
  mocha test describe
    √ mocha test it


  1 passing (10ms)
```

代码中的 describe 和 it 是什么意思呢？其实就是 BDD 中的那些意思，把它们当做语法来记就好了。describe 中的字符串，用来描述你要测的主体是什么；it 当中，描述具体的 case 内容。
我们可以看到，上面的测试代码中什么都没有运行，结果并没有报错，那在什么情况下回报错呢？

> if it throws an Error, it will work!

**_当 `it` 抛出错误的时候，就会产生作用。_**

我们尝试一下：

```javascript
// unit.test.js
describe("mocha test describe", () => {
  it("mocha test it success", () => {});

  it("mocha test it error", () => {
    throw new Error("在 it 中抛出错误！");
  });
});
```

执行测试结果为：

```
  mocha test describe
    √ mocha test it success
    1) mocha test it error


  1 passing (20ms)
  1 failing

  1) mocha test describe mocha test it error:
     Error: 在 it 中抛出错误！
```

相信到这里，你已经对 Mocha 有大致了解。

Mocha 还提供了几种钩子函数 `before()`, `after()`, `beforeEach()`, 和 `afterEach()`。什么是钩子函数呢？用一句话来形容一下：_钩子是将需要执行的函数或者其他一系列动作注册到一个统一的入口，程序通过调用这个钩子来执行这些已经注册的函数_。还是以上面的代码举例：

```javascript
// unit.test.js
describe("mocha test describe", () => {
  // 钩子函数，就是在函数进程中的某个时机被触发的函数
  // before 是一个钩子函数，可以把需要在所有测试前要做的事情写在这里
  before(() => {
    console.info("(before hooks)");
  });

  // beforeEach 也是一个钩子函数，可以把需要在每个测试前都要做的事情写在这里
  beforeEach(() => {
    console.info("(before each hooks)");
  });

  it("mocha test it success", () => {});

  it("mocha test it error", () => {
    throw new Error("在 it 中抛出错误！");
  });
});
```

结果输出：

```
  mocha test describe
  (before hooks)
    (before each hooks)
    √ mocha test it success
    (before each hooks)
    1) mocha test it error


  1 passing (27ms)
  1 failing

  1) mocha test describe mocha test it error:
     Error: 在 it 中抛出错误！
```

Mocha 的介绍就到这里，Mocha 这一块还有关于异步、定时等等一些列的知识，想了解更多可以参考上面链接里的官网地址。

---

# 三、断言库 - should.js

上面在 Mocha 中有提到过，`it` 函数在抛出错误的时候起作用，即发现错误。那么除了手动地抛出错误，还有什么更优雅的方式呢？这就涉及到现在要讲到的 `断言库`。相当于制定了一套判断规则，判断逻辑是否成立。
NodeJs 就内置了一套断言库，可以通过 `require('assert')` 来调用。但我要介绍的是 `should.js`。

> should.js 的 API 文档：[http://shouldjs.github.io/](http://shouldjs.github.io/)

举个例子，我现在要实现一个简单模块，满足一下功能：

- 完成求绝对值的功能
- 输入为数值
- 数值不大于 100
- 数值不小于 -50

为了满足以上条件，我们编写了用例测试代码：

```javascript
// unit.test.js
var absolute = require("../unittest.js").absolute;
const should = require("should");

describe("absolute module test", () => {
  it("when n = 5, it should return 5", () => {
    absolute(5).should.equal(5);
  });

  it("when n = -5, it should return 5", () => {
    absolute(-5).should.equal(5);
  });

  it("when n is not a number, it should throw", () => {
    (function() {
      absolute("string");
    }.should.throw("n should be a Number"));
  });

  it("when n > 100, it should throw", () => {
    (function() {
      absolute(150);
    }.should.throw("n should <= 100"));
  });

  it("when n< -50, it should throw", () => {
    (function() {
      absolute(-50);
    }.should.throw("n should >= -50"));
  });
});
```

如果以下是我们实现的模块：

```javascript
// unit.js
function absolute(n) {
  if (n < 0) {
    return -n;
  }
  return n;
}

exports.absolute = absolute;
```

通过测试我们会得到以下结果：

```
 absolute module test
    √ when n = 5, it should return 5
    √ when n = -5, it should return 5
    1) when n is not a number, it should throw
    2) when n > 100, it should throw
    3) when n< -50, it should throw


  2 passing (27ms)
  3 failing

  1) absolute module test when n is not a number, it should throw:
     AssertionError: expected 'string' to be a function
    expected 'string' to have type function
        expected 'string' to be 'function'

  2) absolute module test when n > 100, it should throw:
     AssertionError: expected 150 to be a function
    expected 150 to have type function
        expected 'number' to be 'function'

  3) absolute module test when n< -50, it should throw:
     AssertionError: expected 150 to be a function
    expected 150 to have type function
        expected 'number' to be 'function'
```

显然没有通过后面三条测试条件，于是我们修改代码 unit.js：

```javascript
// unit.js
function absolute(n) {
  if (typeof n !== "number") {
    throw new Error("n should be a Number");
  }
  if (n < -50) {
    throw new Error("n should >= -50");
  }
  if (n > 100) {
    throw new Error("n should <= 100");
  }
  if (n < 0) {
    return -n;
  }
  return n;
}

exports.absolute = absolute;
```

测试结果：

```
 absolute module test
    √ when n = 5, it should return 5
    √ when n = -5, it should return 5
    √ when n is not a number, it should throw
    √ when n > 100, it should throw
    √ when n< -50, it should throw


  5 passing (19ms)
```

显然通过更改代码后，我们通过所有的用例测试。这就是测试驱动的开发过程。

---

# 四、supertest

在开发 Web 项目的时候，要测试某一个 API，如：/user，到底怎么编写测试用例呢？
使用：supertest

```javascript
var app = require("../app");
var supertest = require("supertest");
// 看下面这句，这是关键一句。得到的 request 对象可以直接按照
// superagent 的 API 进行调用
var request = supertest(app);

var should = require("should");

describe("test/app.test.js", function() {
  // 我们的第一个测试用例，好好理解一下
  it("should return 55 when n is 10", function(done) {
    // 之所以这个测试的 function 要接受一个 done 函数，是因为我们的测试内容
    // 涉及了异步调用，而 mocha 是无法感知异步调用完成的。所以我们主动接受它提供
    // 的 done 函数，在测试完毕时，自行调用一下，以示结束。
    // mocha 可以感知到我们的测试函数是否接受 done 参数。js 中，function
    // 对象是有长度的，它的长度由它的参数数量决定
    // (function (a, b, c, d) {}).length === 4
    // 所以 mocha 通过我们测试函数的长度就可以确定我们是否是异步测试。

    request
      .get("/fib")
      // .query 方法用来传 querystring，.send 方法用来传 body。
      // 它们都可以传 Object 对象进去。
      // 在这里，我们等于访问的是 /fib?n=10
      .query({
        n: 10
      })
      .end(function(err, res) {
        // 由于 http 返回的是 String，所以我要传入 '55'。
        res.text.should.equal("55");

        // done(err) 这种用法写起来很鸡肋，是因为偷懒不想测 err 的值
        // 如果勤快点，这里应该写成
        /*
                should.not.exist(err);
                res.text.should.equal('55');
                */
        done(err);
      });
  });

  // 下面我们对于各种边界条件都进行测试，由于它们的代码雷同，
  // 所以我抽象出来了一个 testFib 方法。
  var testFib = function(n, statusCode, expect, done) {
    request
      .get("/fib")
      .query({
        n: n
      })
      .expect(statusCode)
      .end(function(err, res) {
        res.text.should.equal(expect);
        done(err);
      });
  };
  it("should return 0 when n === 0", function(done) {
    testFib(0, 200, "0", done);
  });

  it("should equal 1 when n === 1", function(done) {
    testFib(1, 200, "1", done);
  });

  it("should equal 55 when n === 10", function(done) {
    testFib(10, 200, "55", done);
  });

  it("should throw when n > 10", function(done) {
    testFib(11, 500, "n should <= 10", done);
  });

  it("should throw when n < 0", function(done) {
    testFib(-1, 500, "n should >= 0", done);
  });

  it("should throw when n isnt Number", function(done) {
    testFib("good", 500, "n should be a Number", done);
  });

  // 单独测试一下返回码 500
  it("should status 500 when error", function(done) {
    request
      .get("/fib")
      .query({
        n: 100
      })
      .expect(500)
      .end(function(err, res) {
        done(err);
      });
  });
});
```

其中 app.js 为：

```javascript
var express = require("express");

// 与之前一样
var fibonacci = function(n) {
  // typeof NaN === 'number' 是成立的，所以要判断 NaN
  if (typeof n !== "number" || isNaN(n)) {
    throw new Error("n should be a Number");
  }
  if (n < 0) {
    throw new Error("n should >= 0");
  }
  if (n > 10) {
    throw new Error("n should <= 10");
  }
  if (n === 0) {
    return 0;
  }
  if (n === 1) {
    return 1;
  }

  return fibonacci(n - 1) + fibonacci(n - 2);
};
// END 与之前一样

var app = express();

app.get("/fib", function(req, res) {
  // http 传来的东西默认都是没有类型的，都是 String，所以我们要手动转换类型
  var n = Number(req.query.n);
  try {
    // 为何使用 String 做类型转换，是因为如果你直接给个数字给 res.send 的话，
    // 它会当成是你给了它一个 http 状态码，所以我们明确给 String
    res.send(String(fibonacci(n)));
  } catch (e) {
    // 如果 fibonacci 抛错的话，错误信息会记录在 err 对象的 .message 属性中。
    // 拓展阅读：https://www.joyent.com/developers/node/design/errors
    res.status(500).send(e.message);
  }
});

// 暴露 app 出去。module.exports 与 exports 的区别请看《深入浅出 Node.js》
module.exports = app;

app.listen(3000, function() {
  console.log("app is listening at port 3000");
});
```

> supertest 官网：[https://github.com/visionmedia/supertest](https://github.com/visionmedia/supertest)
> supertest 的 API 与 superagent 相同：[http://visionmedia.github.io/superagent/](http://visionmedia.github.io/superagent/)
