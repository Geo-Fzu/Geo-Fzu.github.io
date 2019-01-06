---
title: Jade 在 Express 中的使用
date: 2016-07-23 09:32:47
tags: [NodeJs, Jade, Express]
categories: tech
---

Jade 是一个高性能的模板引擎，它是用 JavaScript 实现的，并且可以供 Node 使用。

<!--more-->

1. ## 在 Express 使用 Jade 模板引擎

   ```js
   app.set("views", views);
   // views 为 Jade 文件的存放目录，例如：var views = path.join(__dirname, 'view')
   app.set("view engine", "jade");

   app.get("/", (req, res) => {
     var name = "chiaki";
     var books = ["A", "B", "C"];
     // 在 views 目录下查找 basics 文件，传输数据，并将渲染结果通过 res 返回页面
     res.render("basics", {
       name: name,
       books: books
     });
   });
   ```

2. ## 一些基本语法：

   - **标签**

     - 标签就用一个简单的单词：
       例如 html 标签写作： `html`
       渲染后再页面中会被转换为标签：
       ```html
       <html></html>
       ```
     - 加 class 和 id 的标签：
       ```jade
       div#container
       div.bar
       div.#container.bar.baz
       ```
       渲染后分别被转换为：
       ```html
       <div id="container"></div>
       <div class="bar"></div>
       <div class="bar baz" id="container"></div>
       ```
     - 标签的文本内容，直接放在标签之后：
       `p hello,chiaki`
       转化成：
       ```html
       <p>hello,chiaki</p>
       ```
     - 文本内容从变量中取值：

       ```jade
       h1 foo
       h2= book.name
       h3 "#{book.name}" for #{book.price} €

       // {"book": {"name": "Hello", "price": 12.99}}
       ```

       转化成：

       ```html
       <h1>foo</h1>
       <h2>Hello</h2>
       <h3>"Hello" for 12.99 €</h3>
       ```

   - **注释**
     - 单行注释和 JavaScript 里是一样的，通过 `//` 来开始，并且**必须单独一行**：
       ```jade
       // just some paragraphs
       p foo
       p bar
       ```
       渲染为：
       ```html
       <!-- just some paragraphs -->
       <p>foo</p>
       <p>bar</p>
       ```
       Jade 同样支持不输出的注释，加一个短横线就行了：
       ```jade
       //- will not output within markup
       p foo
       p bar
       ```
       渲染为：
       ```html
       <p>foo</p>
       <p>bar</p>
       ```
   - **内联**
     通过缩进的方式内联：
     ```jade
     ul
       li.first
         a(href='#') foo
       li
         a(href='#') bar
       li.last
         a(href='#') baz
     ```
     渲染为：
     ```html
     <ul>
       <li class="first"><a href="#">foo</a></li>
       <li><a href="#">bar</a></li>
       <li class="last"><a href="#">baz</a></li>
     </ul>
     ```
   - **属性**
     元素的属性写在 `( )` 中，当一个值是 `undefined` 或者 `null` 属性 不 会被加上：
     ```jade
     a(href='/login', title='View login page') Login
     ```
   - **代码**

     ```jade
     // 使用 `-` 前缀，这是不会被输出的：
     - var foo = 'bar';

     //  可以用在循环中：
     - for (var key in obj)
       p= obj[key]

     // 由于 Jade 缓存技术，下面的代码也是可以的：
     - if (foo)
       ul
         li yay
         li foo
         li worked
     - else
       p oh no! didnt work

     // 或者：
     - if (items.length)
       ul
         - items.forEach(function(item){
           li= item
         - })
     ```

   - **循环**
     尽管已经支持 JavaScript 原生代码，Jade 还是支持了一些特殊的标签，它们可以让模板更加易于理解，其中之一就是 `each`, 这种形式：

     ```jade
     each VAL[, KEY] in OBJ
     ```

     或者使用 `for`：

     ```jade
     for VAL in OBJ
     ```

     如果使用 JavaScript 原生代码：

     ```jade
     - books.forEach(function (book, i, arr) {
         li book: #{book}
     - })

     - for(i in books){
         li book: #{books[i]}
     - }
     ```

3. ## 模板继承

   Jade 支持通过 `block` 和 `extends` 关键字来实现模板继承。一个块就是一个 Jade 的 block ，它将在子模板中实现，同时是支持递归的。Jade 的模板类似 Java 中的接口，只不过前者继承的是视图和模板，后者继承的是逻辑和方法。
   Jade 块如果没有内容，Jade 会添加默认内容，下面的代码默认会输出 block scripts, block content, 和 block foot。

   ```jade
   // layout.jade
   html
     head
       h1 My Site - #{title}
       block scripts
         script(src='/jquery.js')
     body
       block content
       block foot
         #footer
           p some footer content
   ```

   现在我们来继承这个布局，简单创建一个新文件，像下面那样直接使用 `extends`，给定路径（可以选择带 .jade 扩展名或者不带）。你可以定义一个或者更多的块来覆盖父级块内容, 注意到这里的 foot 块没有定义，所以它还会输出父级的 "some footer content"。

   ```jade
   extends layout

   block scripts
     script(src='/jquery.js')
     script(src='/pets.js')

   block content
     h1= title
     each pet in pets
       include pet
   ```

   Jade 允许你`替换（默认）`、`前置`和`追加` blocks. 比如，假设你希望在所有页面的头部都加上默认的脚本，你可以这么做：

   ```jade
   // layout.jade
   html
     head
       block head
         script(src='/vendor/jquery.js')
         script(src='/vendor/caustic.js')
     body
       block content
   ```

   现在假设你有一个 Javascript 游戏的页面，你希望在默认的脚本之外添加一些游戏相关的脚本，你可以直接 `append` 上代码块：

   ```jade
   extends layout

   block append head
     script(src='/vendor/three.js')
     script(src='/game.js')
   ```

   使用 block append 或 block prepend 时 `block` 是可选的:

   ```jade
   extends layout

   append head
     script(src='/vendor/three.js')
     script(src='/game.js')
   ```

4. ## Mixins

   Mixins 在编译的模板里会被 Jade 转换为普通的 JavaScript 函数。 Mixins 可以带参数，但不是必需的：

   ```jade
   mixin list
     ul
       li foo
       li bar
       li baz
   ```

   使用不带参数的 mixin 看上去非常简单，在一个块外：

   ```jade
   h2 Groceries
   mixin list
   ```

   Mixins 也可以带一个或者多个参数，参数就是普通的 JavaScript 表达式，比如下面的例子：

   ```jade
   mixin pets(pets)
     ul.pets
       + each pet in pets
         li= pet

   mixin profile(user)
     .user
       h2= user.name
       mixin pets(user.pets)
   ```

   会输出像下面的 HTML:

   ```html
   <div class="user">
     <h2>tj</h2>
     <ul class="pets">
       <li>tobi</li>
       <li>loki</li>
       <li>jane</li>
       <li>manny</li>
     </ul>
   </div>
   ```
