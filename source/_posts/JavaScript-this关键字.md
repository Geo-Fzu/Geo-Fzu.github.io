---
title: JavaScript this关键字
date: 2016-07-19 11:09:38
tags: [JavaScript, 概念]
categories: tech
---

`this`是 Javascript 语言的一个关键字,它代表函数运行时，自动生成的一个内部对象，只能在函数内部使用。随着函数使用场合不同，`this`的值会发生变化。但有一个总的原则，那就是`this`指的是**调用函数的那个对象**。

<!--more-->

下面分五种情况，详细讨论`this`的用法。

1. ### 纯粹的函数调用：
   这是函数最通常的用法，属于全局性调用，因此`this`就代表全局对象 Global（浏览器中的 window）。
   - 示例：
     ```js
     var x = 0;
     function test() {
       console.log(this.x);
     }
     test();
     // test 在全局作用域中，相当于 Global.test(), this.x 指向 Global.x。
     // 这里输出结果为：0。
     ```
2. ### 作为对象方法调用：
   函数还可以作为某个对象的方法调用，这时`this`就指这个上级对象,即调用该方法的对象。
   - 示例：
     ```js
     function test() {
       console.log(this.x);
     }
     var o = {};
     o.x = 1;
     o.f = test;
     o.f();
     // test 作为 o 的对象方法,由 o 对象来调用, this.x 指向 o.x。
     // 这里输出结果为：1。
     ```
3. ### 在构造函数中调用：
   构造函数，就是通过这个函数生成一个新对象 object。这时，this 就指这个新对象。
   - 示例：
     ```js
     function Foo() {
       this.x = 1;
     }
     var o = new Foo();
     console.log(o.x);
     // this 在构造函数 Foo 中调用，指向由 Foo 创建的对象（o）。
     // 这里输出结果为：1。
     ```
4. ### 在 apply 和 call 中调用：
   Function.prototype.apply()是函数对象的一个方法，它的作用是改变函数的调用对象，它的第一个参数就表示改变后的调用这个函数的对象。
   - 示例：
     ```js
     var x = 0;
     function test() {
       console.log(this.x);
     }
     var o = {};
     o.x = 1;
     o.m = test;
     o.m.apply(); //这里输出结果为：0。
     o.m.apply(o); //这里输出结果为：1。
     // apply()的参数为空时，默认调用全局对象。因此，第一次的运行结果为0，证明this指的是全局对象。
     // apply(o), this 指向 o, this.x 指向 o.x,所以输出结果为 1.
     ```
5. ### 箭头函数中的 this：
   箭头函数是 ES6 中的新特性，函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象。
   - 示例：
     ```js
     //比如Mongoose 定义 Schema 的 statics/methods 的时候:
     XXXSchema.statics.list = cb => {
       return this.find().exec(cb);
     };
     // 以上函数不能达到想要的效果，因为 this 并不是指向由 XXX Model 创建的 documents，而是指向定义时所在的对象。应该改为以下形式：
     XXXSchema.statics.list = function(cb) {
       return this.find().exec(cb);
     };
     ```
