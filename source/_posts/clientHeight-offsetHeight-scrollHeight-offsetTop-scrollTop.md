---
title: clientHeight/offsetHeight/scrollHeight/offsetTop/scrollTop
date: 2019-01-08 01:20:59
tags: [JavaScript, CSS, 概念]
categories: tech
---

clientHeight, offsetHeight, scrollHeight, offsetTop 以及 scrollTop

<!--more-->

![图示](http://cdn.upon.ink/image/scroll-top-height-illustrator.png "高度及位置")

## clientHeight

`Element.clientHeight` 是只读属性。 如果是 `inline` 元素其值为 0；否则其值为 content 高度加上 padding 距离，如果有滚动条就减去滚动条宽度（不包含 border, margin 距离）。

Note: 这个属性值是四舍五入为整数，如果需要小数值，用 `Element.getBoundingClientRect()`。

## offsetHeight

`Element.offsetHeight` 是只读属性。包含 padding 和 border, 是一个整数。offsetHeight 是具有代表性的元素高度的测量方法，不包含类似 _::after_ 和 _::before_ 的伪元素的高度。如果元素被隐藏（例如设置了 display: none), 其值为 0。

## scrollHeight

`Element.scrollHeigh` 是只读属性，用来测量元素内容的高度，包括因为 overflow 没有展示在屏幕上的那部分。

换句话说，scrollHeight 是该元素呈现在屏幕上不需要出现滚动条而需要的最小高度。包括 padding （不包含 border, margin 距离），也包含 _::after_ 和 _::before_ 的高度。**在不需要垂直滚动条的情况下，scrollHeight 与 clientHeight 值相等**。

## offsetTop

`Element.offsetTop` 是只读属性，表示当前元素距离 `offsetParent` 顶部的距离。

_offsetParent_ 是与当前元素最近的经过定位（position 不等于 static）的父级元素。

## scrollTop

`Element.scrollTop` 可以读取和设置元素内容在垂直方向上滚动的距离，即总 top 与可视部分 top 之间距离。如果没有产生垂直滚动条，srcollTop 值就是 0.

## 使用场景

1. 判断滑动到到底部，比如 `加载更多功能`：Element.scrollTop + Element.clientHeight == Element.scrollHeight
