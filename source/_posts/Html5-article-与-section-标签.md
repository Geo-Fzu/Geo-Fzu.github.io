---
title: Html5 article 与 section 标签
date: 2016-07-24 20:04:49
tags: [Html5, 标签, 语义化]
categories: tech
---
# article
article 标签，从语义化上看为文档、页面，其用法如下：
* 通常是一篇文章、一个页面、一个独立完整的内容模块
* 一般会带个标题，并放在 `header` 标签中
* article 元素可以互相嵌套
<!--more-->

**使用频率极高，强调独立性，多注意下与  `header` 标签的使用。**
``` html
<article>
  <header>
    <h1>标题</h1>
  </header>
  <p>我是段落</p>
  <article>
    <div>我的内容</div>
  </article>
</article>
```
article 元素代表文档、页面或应用程序中独立的、完整的、可以独自被外部引用的内容。它可以是一篇博客或报刊中的文章、一篇论坛帖子、一段用户评论或独立的插件，或其他任何独立的内容。除了内容部分，*一个 article 元素通常有它自己的标题*（一般放在一个header元素里面），有时还有自己的脚注。
``` html
<article>
  <header>
    <h1>article元素使用方法</h1>
    <p>
      发表日期：
      <time pubdate="pubdate">2016/07/24</time>
    </p>
  </header>
  <p>article元素是什么？怎样使用article元素？……</p>
  <footer>
    <p>
      <small>Copyright @Tingandpeng All Rights Reserverd</small>
    </p>
  </footer>
</article>
```
关于article元素嵌套的代码示例，代码如下：
``` html
<article>
  <header>
    <h1>article元素使用方法</h1>
    <p>
      发表日期：
      <time pubdate="pubdate">2016/07/24</time>
    </p>
  </header>
  <p>article元素是什么？怎样使用article元素？……</p>
  <section>
    <h2>评论</h2>
    <article>
      <header>
        <h3>发表者：Chiaki</h3>
        <p>
          <time pubdate datetime="2016-07-24T:21-26:00">1小时前</time>
        </p>
      </header>
      <p>这篇文章很不错啊，顶一下！</p>
    </article>
    <article>
      <header>
        <h3>发表者：Takai</h3>
        <p>
          <time pubdate datetime="2016-07-24T:21-26:00">1小时前</time>
        </p>
      </header>
      <p>这篇文章很不错啊，对article解释的很详细</p>
    </article>
  </section>
</article>
```
# section
section 标签，从语义化上看为部分，其用法如下：
* 用于页面内容的独立分块，往往是文章的一段
* 如果article元素、aside元素、nav元素更符合使用条件，那不要使用section元素
* 通常由内容和标题组成，没有标题的内容不推荐使用 section

**使用频率低，强调分段分块。**
*注：《HTML5与CSS3权威指南》这本书中说明：一个容器需要被定义样式或者脚本定义行为时，推荐用div而非section，不要将section用作设置样式的容器。*
``` html
<section>
  <h1>水果</h1>
  <article>
    <h2>苹果</h2>
    <div>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Soluta dolor iure, sapiente fugiat ipsa numquam ad et. Nostrum quibusdam voluptatibus vero eligendi placeat. Cumque consectetur velit, nobis ipsa suscipit dicta.</div>
  </article>
  <article>
    <h2>桔子</h2>
    <div>Lorem ipsum dolor sit amet, consectetur adipisicing elit. Tempora dolorem in officia quo quam vel hic maiores consectetur, nulla a voluptatum nobis dolores amet debitis, aperiam odit, eius. Deserunt, dolorum.</div>
  </article>
</section>
<!-- article可以看成是一种特殊种类的section元素,它比section更强调独立性 -->
<article>
  <h1>中国人物</h1>
  <p>三国、两晋、南北朝</p>
  <section>
    <h2>三国</h2>
    <p>猛将猛将猛将猛将</p>
  </section>
  <section>
    <h2>两晋</h2>
    <p>猛将猛将猛将猛将</p>
  </section>
</article>
```
**以上讲述了那么多，两者的区别到底是什么呢？事实上，在HTML5中，article元素可以看成是一种特殊类型的section元素，它比section元素更强调独立性。即section元素强调分段或分块，而article强调独立性。具体来说，如果一块内容相对来说比较独立的、完整的时候，应该使用article元素，但是如果你想将一块内容分成几段的时候，应该使用section元素。另外，在HTML5中，div元素变成了一种容器，当使用CSS样式的时候，可以对这个容器进行一个总体的CSS样式的套用。**