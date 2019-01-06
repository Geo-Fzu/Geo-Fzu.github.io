---
title: ES6 函数的扩展
date: 2016-07-19 08:58:09
tags: [JavaScript, ES6]
categories: tech
<!-- comments: false -->
---

- <a href = "#arrowFunc">箭头函数</a>
- <a href = "#dfltVal">函数参数的默认值</a>
- <a href = "#rest">rest 参数</a>
- <a href = "#extnOpr">扩展运算符</a>
- <a href = "#name">name 属性</a>
- <a href = "#funcBind">函数绑定</a>
- 尾调用优化
- 函数参数的尾逗号
  <!--more-->

1. ## <a id = "arrowFunc">箭头函数</a>

   - ### 基本用法
     ES6 允许使用“箭头”(=>)定义函数。

   ```js
   var f = v => v;
   ```

   上面的箭头函数等同于：

   ```js
   var f = function(v) {
     return v;
   };
   ```

   如果箭头函数不需要参数或需要多个参数，就使用一个圆括号代表参数部分。

   ```js
   var f = () => 5;
   //等同于
   var f = function() {
     return 5;
   };

   var sum = (num1, num2) => num1 + num2;
   //等同于
   var sum = function(num1, num2) {
     return num1 + num2;
   };
   ```

   如果箭头函数的代码块部分多于一条语句，就要使用大括`{}`将它们括起来，并且使`return`语句返回。

   ```js
   var sum = (num1, num2) => {
     return num1 + num2;
   };
   ```

   由于大括号被解释为代码块，所以如果箭头函数直接返回一个对象，必须在外面加上括号。

   ```js
   var getTempObj = id => ({
     id: id,
     name: "temp"
   });
   ```

   - ### 使用注意点
     **箭头函数有几个使用注意点。**

   1. 函数体内的`this`对象，就是定义时所在的对象，而不是使用时所在的对象。
   2. 不可以当做构造函数，也就是说，不可以使用`new`命令，否则会抛出一个错误。
   3. 不可以使用`arguments`对象，该对象在函数体内不存在。如果要用，可以使用 Rest 参数代替。
   4. 不可以使用`yield`命令，因此箭头函数不能用作 Generator 函数。

2. ## <a id = "dfltVal">函数参数的默认值</a>

   - ### 基本用法
     ES6 允许为函数的参数设置默认值，即直接写在参数定义的后面。

   ```js
   function log(x, y = "World") {
     console.log(x, y);
   }

   log("Hello"); // Hello World
   log("Hello", "Chiaki"); // Hello Chiaki
   log("Hello", ""); // Hello
   ```

   另一个例子：

   ```js
   function Point(x = 0, y = 0) {
     this.x = x;
     this.y = y;
   }

   var p = new Point();
   console.log(p); // Point { x: 0, y: 0 }
   ```

   ES6 函数参数默认值写法有以下好处：
   _ 简洁
   _ 易于阅读，可以轻易意识到哪些参数是可以省略的 \* 有利于将来代码优化，即使在对外接口中拿掉这个参数也照常运行

   - ### 与解构赋值默认值结合使用
     参数默认值可以与解构赋值的默认值，结合起来使用。

   ```js
   function foo({ x, y = 5 }) {
     console.log(x, y);
   }

   foo({}); // undefined 5
   foo({ x: 1 }); // 1 5
   foo({ x: 1, y: 2 }); // 1 2
   foo(); // error
   ```

   上面的代码使用了对象的解构赋值默认值，而没有使用函数参数的默认值。只有当函数`foo`的参数是一个对象时，变量`x`和`y`才会通过解构赋值产生。如果函数`foo`调用时参数不是对象，变量`x`和`y`就不会产生。
   下面是另一个对象的解构赋值默认值的例子。

   ```js
   function fetch(url, { body = "", method = "GET", headers = {} }) {
     console.log(method);
   }

   fetch("http://example.com", {});
   // "GET"

   fetch("http://example.com");
   // 报错
   ```

   - ### 作用域
     如果参数默认值是一个变量，则该变量所处的作用域，与其他变量的作用域规则是一样的，即先是当前函数的作用域，然后才是全局作用域。

   ```js
   var x = 1;

   function f(x, y = x) {
     console.log(y);
   }

   f(2); // 2
   ```

   上面代码中，参数 y 的默认值等于 x。调用时，由于函数作用域内部的变量 x 已经生成，所以 y 等于参数 x，而不是全局变量 x。

3. ## <a id = "rest">rest 参数</a>

   - ### 基本用法
     ES6 引入 rest 参数（形式为“...变量名”），用于获取函数的多余参数，这样就不需要使用 arguments 对象了。rest 参数搭配的变量是一个数组，该变量将多余的参数放入数组中。

   ```js
   function add(...values) {
     let sum = 0;

     for (var val of values) {
       sum += val;
     }

     return sum;
   }

   add(2, 3, 4); // 9

   // 该 add 函数是一个求和函数，利用 rest 参数，可以向该函数传入任意数目的参数。
   ```

   下面是一个 rest 参数代替 arguments 变量的例子：

   ```js
   // arguments 变量的写法
   function sortNumbers() {
     return Array.prototype.slice.call(arguments).sort();
   }

   // rest 参数的写法
   const sortNumbers = (...numbers) => numbers.sort();
   ```

   **注意， rest 参数之后不能再有其它参数（即只能是最后一个参数），否则会报错。**

   ```js
   function (a, ...b, c)
       // ...
   }
   // error
   ```

4. ## <a id = "extnOpr">扩展运算符</a>

   - ### 基本用法
     扩展运算符（spread）是三个点（...）。它好比 rest 参数的逆运算，将一个数组转为用逗号分隔的参数序列。

   ```js
   console.log(...[1, 2, 3])
   // 1 2 3

   console.log(1, ...[2, 3, 4], 5)
   // 1 2 3 4 5

   [...document.querySelectorAll('div')]
   // [<div>, <div>, <div>]
   ```

   该运算符主要用于函数调用：

   ```js
   // 第一个示例：
   function push(array, ...items) {
     array.push(...items);
   }

   // 第二个示例：
   function add(x, y) {
     return x + y;
   }

   var numbers = [4, 38];
   add(...numbers); // 42
   ```

   **分析：**上面代码中，函数定义时的参数 如上面 push(array, ...items) 中的 `...items` 是 rest 参数，见上一小节，用于获取函数的多余参数。函数调用时，如代码中的 array.push(...items)， `...items` 是扩展运算符的应用，将数组转为参数序列。

5. ## <a id = "name">name 属性</a>

   - ### 基本用法
     函数的 name 属性，返回该函数的函数名。

   ```js
   function foo() {}
   console.log(foo.name);
   // "foo"
   ```

   这个属性早就被浏览器广泛支持，但是直到 ES6， 才将其写入了标准。

   需要注意的是， ES6 对这个属性的行为做出了一些修改。如果将一个匿名函数赋值给一个变量， ES5 的 name 属性，会返回空字符串，而 ES6 的 name 属性会返回实际的函数名：

   ```js
   var func1 = function() {};

   // ES5
   func1.name; // ""

   // ES6
   func1.name; // "func1"
   ```

   如果将一个具名函数赋值给一个变量，则 ES5 和 ES6 的 name 属性都返回这个具名函数原本的名字：

   ```js
   const bar = function baz() {};

   // ES5
   bar.name; // "baz"

   // ES6
   bar.name; // "baz"
   ```

   Function 构造函数返回的函数实例，name 属性的值为“anonymous”：

   ```js
   new Function().name; // "anonymous"
   ```

   bind 返回的函数，name 属性值会加上“bound ”前缀：

   ```js
   function foo() {}
   foo
     .bind({})
     .name(
       // "bound foo"

       function() {}
     )
     .bind({}).name; // "bound "
   ```

6. ## <a id = "funcBind">函数绑定</a>

   - ### 基本用法
     箭头函数可以绑定 this 对象，大大减少了显式绑定 this 对象的写法（call、apply、bind）。但是，箭头函数并不适用于所有场合，所以 ES7 提出了“函数绑定”（function bind）运算符，用来取代 call、apply、bind 调用。虽然该语法还是 ES7 的一个提案，但是 Babel 转码器已经支持。

   函数绑定运算符是并排的两个双冒号（::），双冒号左边是一个对象，右边是一个函数。该运算符会自动将左边的对象，作为上下文环境（即 this 对象），绑定到右边的函数上面：

   ```js
   foo::bar;
   // 等同于
   bar.bind(foo);

   foo::bar(...arguments);
   // 等同于
   bar.apply(foo, arguments);

   const hasOwnProperty = Object.prototype.hasOwnProperty;
   function hasOwn(obj, key) {
     return obj::hasOwnProperty(key);
   }
   ```

   如果双冒号左边为空，右边是一个对象的方法，则等于将该方法绑定在该对象上面：

   ```js
   var method = obj::obj.foo;
   // 等同于
   var method = ::obj.foo;

   let log = ::console.log;
   // 等同于
   var log = console.log.bind(console);
   ```

> [参考链接](http://es6.ruanyifeng.com/#docs/function "ECMAScript 6 入门")
