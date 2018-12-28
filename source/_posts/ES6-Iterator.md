---
title: ES6 Iterator
date: 2016-08-04 15:16:06
tags: [ES6, JavaScript, 概念]
categories: tech
---

# 一、 Iterator 的概念

迭代器（Iterator）是一个接口，为各种不同的数据结构提供统一的访问机制。任何数据只要部署了 `Iterator` 接口，就可以完成遍历操作。
迭代器的作用有：

- 为各种数据结构提供一个统一的、简便的访问接口；
- 使数据结构成员能够按某种次序排列；
- ES6 创造了一种新的遍历命令 `for...of` 循环，Iterator 接口主要供 for...of 消费。
  <!--more-->

就拿 for...of 语句来说，它首先调用被遍历对象的 `[Symbol.iterator]()` 方法，该方法返回一个迭代器对象，**迭代器对象可以是拥有 `next` 方法的任何对象**。然后， 在 for...of 的每一次循环中，都将调用该迭代器对象上的 next 方法。
每一次调用 next 方法，都会返回数据结构的当前成员信息。具体来说就是返回一个包含 value 和 done 两个属性的对象。其中 value 是当前成员的值，done 是一个布尔值，表示遍历是否结束。

下面的代码实现了一个简单的迭代器对象：

```javascript
var sampleIterator = {
  index: 0,
  [Symbol.iterator]: function() {
    return this;
  },
  next: function() {
    if (this.index < 3) {
      return {
        done: false,
        value: this.index++
      };
    } else {
      return {
        done: true,
        value: undefined
      };
    }
  }
};

for (var val of sampleIterator) {
  console.log(val);
}

// 结果为：
// 0
// 1
// 2
// [Finished in 2.7s]
```

上面的代码中，当使用 for...of 遍历 sampleIterator 时，首先调用了该对象的 `[Symool.itirator]` 方法，该方法返回对象本身。而该对象中包含有 `next` 方法，所以该对象本身就是一个 Iterator 对象。可以供 for..of 消费。当 this.index >= 3 时，返回 `{done: true, value: undefined}`, 循环结束。

# 二、 数据结构的默认 Iterator 接口

Iterator 接口的目的，就是为所有的数据结构提供一种统一的访问机制，即 `for...of` 循环。当使用 for...of 循环遍历某种数据结构时，该循环会自动去寻找 Iterator 接口。
ES6 规定，默认的 Iterator 接口就部署在数据结构的 `Symbol.iterator` 属性。调用该方法，就会得到当前数据结构默认的迭代器生成函数。
ES6 中，有三类数据结构原生具备 `Iterator` 接口：数组、类似数组的对象（如 NodeList ）、Set 和 Map 结构。

```javascript
let arr = [1, 2, 4];

// 迭代器接口部署在数组的 Symbol.iterator 属性上，调用该属性就可以得到迭代器对象（一个包含 next 函数的对象）
var iterator = arr[Symbol.iterator]();

console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());
console.log(iterator.next());

// 输出结果：
// { value: 1, done: false }
// { value: 2, done: false }
// { value: 4, done: false }
// { value: undefined, done: true }
// [Finished in 2.6s]
```

类似数组的对象（_存在数值键名和 length 属性_），可以直接在 Symbol.iterator 属性上部署数组的 Iterator 接口：

```javascript
let iterable = {
  0: "a",
  1: "b",
  2: "c",
  3: "d",
  length: 4,
  [Symbol.iterator]: Array.prototype[Symbol.iterator]
};

for (var item of iterable) {
  console.log(item);
}

// 输出结果：
// a
// b
// c
// d
// [Finished in 2.6 s]

// 如果 iterable 的 length 属性为 3：
// a
// b
// c
// [Finished in 2.7s]

// 如果 iterable 的 length 属性为 5：
// a
// b
// c
// d
// undefined
// [Finished in 2.7s]
```

**注：**普通对象部署数组的 Symbol.iterator 方法，并没有效果。如果 Symbol.iterator 方法对应的不是遍历器生成函数，解释引擎会报错。

# 三、 调用 Iterator 接口的场合

除了 for...of 循环，还有几个场合会默认调用 Iterator 接口（即 Symbol.iterator).

1. 解构赋值
2. 扩展运算符
3. yield\*
4. 由于数组遍历调用迭代器接口，所以任何接受数组作为参数的场合，其实都调用了 Iterator 接口：
   - for...of
   - Array.from()
   - Map(), Set(), WeakMap(), WeakSet()
   - Promise.all()
   - Promise.race()
   - ...

# 四、 for...of 与其它遍历语法比较

- for 循环

```javascript
for (var i = 0; i < arr.length; i++) {
  console.log(arr[i]);
}

// 这种方法比较麻烦
```

- forEach

```javascript
arr.forEach(function(item) {
  console.log(item);
});

// 这种方法无法中途跳出 forEach 循环，break 或 return 失效
```

- for...in

```javascript
for (let i in arr) {
  console.log(arr[i]);
}
```

    for...in 循环有几个缺点：
    * 以字符串作为键名，而数组的键名是数值类型
    * 不仅遍历数字键名，还会遍历手动添加的其它键
    * 以任意顺序遍历键名
    * 总之，它主要是为了遍历对象设计的，不适用于遍历数组

- for...of 优点
  - 和 for...in 一样简洁，但是没有上面提到的缺点
  - 可以使用 break、 return 和 continue
  - 提供了遍历所有数据结构的统一操作接口
