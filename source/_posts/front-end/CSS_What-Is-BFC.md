---
title: CSS中的 BFC 是什么？
date: 2020-10-27 21:00:00
tags: [CSS, BFC]
categories: CSS
---
BFC: 块级格式化上下文 (Block Formatting Context)
<!--more-->
> 建议先理解CSS的[盒模型](https://chenchenwuai.github.io/front-end/CSS_Box_Model/)

## 什么是BFC
BFC是Web页面的可视CSS渲染的一部分，是块盒子的布局过程发生的区域，也是浮动元素与其他元素交互的区域。W3C对BFC的定义如下：
>浮动元素和绝对定位元素，非块级盒子的块级容器（例如 inline-blocks, table-cells, 和 table-captions），以及overflow值不为“visiable”的块级盒子，都会为他们的内容创建新的BFC（块级格式上下文）。

BFC是一个独立的布局环境，其中的元素布局是不受外界的影响，通俗的说可以把BFC理解为一个封闭的箱子，箱子内部的元素无论怎么放置，都不会影响到外部。并且在一个 BFC 中，块盒与行盒（行盒由一行中所有的内联元素所组成）都会垂直的沿着其父元素的边框排列。

下列方式会创建BFC：

+ 根元素（`<html>）`
+ 浮动元素（元素的 `float` 不是 `none`）
+ 绝对定位元素（元素的 `position` 为 `absolute` 或 `fixed`）
+ 行内块元素（元素的 `display` 为 `inline-block`）
+ 表格单元格（元素的 `display` 为 `table-cell`，HTML表格单元格默认为该值）
+ 表格标题    （元素的 `display` 为 `table-caption`，HTML表格标题默认为该值）
+ `display` 值为 `flow-root` 的元素
+ 匿名表格单元格元素（元素的 `display` 为 `table、``table-row`、 `table-row-group、``table-header-group、``table-footer-group`（分别是HTML table、row、tbody、thead、tfoot 的默认属性）或 `inline-table`）
+ `overflow` 值不为 `visible` 的块元素
+ `contain` 值为 `layout`、`content `或 paint 的元素
+ 弹性元素（`display` 为 `flex` 或 `inline-flex `元素的直接子元素）
+ 网格元素（`display` 为 `grid` 或 `inline-grid` 元素的直接子元素）
+ 多列容器（元素的 [`column-count`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-count) 或 [`column-width`](https://developer.mozilla.org/zh-CN/docs/Web/CSS/column-width) 不为 `auto，包括 ``column-count` 为 `1`）
+ `column-span` 为 `all` 的元素始终会创建一个新的BFC，即使该元素没有包裹在一个多列容器中（[标准变更](https://github.com/w3c/csswg-drafts/commit/a8634b96900279916bd6c505fda88dda71d8ec51)，[Chrome bug](https://bugs.chromium.org/p/chromium/issues/detail?id=709362)）（未验证）。

## BFC的特性

+ BFC内部的块级盒会在垂直方向上一个接一个排列。（普通流）

+ 计算BFC的高度时，浮动元素也会参与计算。

+ 同一BFC下的相邻块级元素可能发生外边距折叠。(float可以避免外边距折叠 或者 打破同一BFC的结构)

+ BFC元素不会和它的子元素发生外边距折叠。

+ 浮动盒的区域不会和BFC重叠。

+ BFC是一个独立的容器外面的元素不会影响BFC内部反之亦然。

## BFC的作用

### 清除内部浮动
未完待续...