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
我们在布局时经常会遇到这个问题：对子元素设置浮动后，父元素会发生高度塌陷，可能出现子元素溢出父元素。例如：
```css
.box {
    background-color: rgb(224, 206, 247);
    border: 5px solid green;
}

.float {
    float: left;
    height: 50px;
    border:1px solid black;
}
```
``` html
<div class="box">
    <div class="float">I am a floated box!</div>
</div> 
```

上面代码的显示效果为：

![](https://ftp.bmp.ovh/imgs/2020/10/de7b28122790a2d2.png)

解决这个问题，只需要把父元素变成一个BFC就行了。常用的办法是给父元素的overflow设置一个不是visible的值，可以是hidden、auto、scroll。父元素就会被撑开。

``` css
.box {
    background-color: rgb(224, 206, 247);
    border: 5px solid green;
    overflow: hidden;
}
```
![](https://ftp.bmp.ovh/imgs/2020/10/fd0909d62441d1f2.png)

另一种方法是设置父元素的`display` 为 `flow-root`,一个新的 `display` 属性的值（所以请注意兼容性问题），它可以创建无副作用的 BFC。关于值 `flow-root`的这个名字，当你明白你实际上是在创建一个行为类似于根元素 （浏览器中的`<html>`元素） 的东西时，就能发现这个名字的意义了——即创建一个上下文，里面将进行 flow layout。

### 消除边距折叠

#### 什么是边距折叠

块的上外边距(margin-top)和下外边距(margin-bottom)有时合并(折叠)为单个边距，其大小为单个边距的最大值(或如果它们相等，则仅为其中一个)，这种行为称为**边距折叠**。

> 注意有设定 float 和 position=absolute 的元素不会产生外边距重叠行为。

有三种情况会形成外边距重叠：

- 1.相邻的两个元素之间的外边距重叠，除非后一个元素加上[clear-fix清除浮动](https://developer.mozilla.org/zh-CN/docs/Web/CSS/clear)。

  ```html
  <style>   
  p:nth-child(1){   
    margin-bottom: 13px; 
  }   
  p:nth-child(2){  
    margin-top: 87px;  
  } 
  </style>
  
  <p>下边界范围会...</p>
  <p>...会跟这个元素的上边界范围重叠。</p>
  ```

  这个例子如果以为边界会合并的话，理所当然会猜测上下2个元素会合并一个100px的边界范围，但其实会发生边界折叠，只会挑选最大边界范围留下，所以这个例子的边界范围其实是87px。

  

- 2.没有内容将父元素和后代元素分开

   如果没有边框border，内边距padding，行内内容，也没有创建块级格式上下文或清除浮动来分开一个块级元素的上边界margin-top 与其内一个或多个后代块级元素的上边界margin-top；或没有边框，内边距，行内内容，高度height，最小高度min-height或 最大高度max-height 来分开一个块级元素的下边界margin-bottom与其内的一个或多个后代后代块元素的下边界margin-bottom，则就会出现父块元素和其内后代块元素外边界重叠，重叠部分最终会溢出到父级块元素外面。 

```html
  <style type="text/css">
      section    {
          margin-top: 13px;
          margin-bottom: 87px;
      }
      header {
          margin-top: 87px;
      }
      footer {
          margin-bottom: 13px;
      }
  </style>
  
  <section>
      <header>上边界重叠 87</header>
      <main></main>
      <footer>下边界重叠 87 不能再高了</footer>
  </section>
```

- 3.空的块级元素

  当一个块元素上边界margin-top 直接贴到元素下边界margin-bottom时也会发生边界折叠。这种情况会发生在一个块元素完全没有设定边框border、内边距paddng、高度height、最小高度min-height 、最大高度max-height 、内容设定为inline或是加上clear-fix的时候。

  ```html
  <style>
  p {
    margin: 0;  
  }
  div {
    margin-top: 13px;
    margin-bottom: 87px;
  }
  </style>
  
  <p>上边界范围是 87 ...</p>
  <div></div>
  <p>... 上边界范围是 87</p>
  ```

一些需要注意的地方：

- 上述情况的组合会产生更复杂的外边距折叠。
- 即使某一外边距为0，这些规则仍然适用。因此就算父元素的外边距是0，第一个或最后一个子元素的外边距仍然会“溢出”到父元素的外面。
- 如果参与折叠的外边距中包含负值，折叠后的外边距的值为最大的正边距与最小的负边距（即绝对值最大的负边距）的和,；也就是说如果有-13px 8px 100px叠在一起，边界范围的技术就是 100px -13px的87px。
- 如果所有参与折叠的外边距都为负，折叠后的外边距的值为最小的负边距的值。这一规则适用于相邻元素和嵌套元素。

#### 怎么消除边距折叠？

对于上面的情况，只需要给他们的父元素创建一个BFC就能消除。



### 创建两边固定中间自适应布局

```html
<style>
.wrap{
  border: 10px pink solid;
  overflow: hidden;
}
.box1{
  width: 100px;
  height: 100px;
  background: red;
  float: left;
}
.box2{
  width: 200px;
  height: 100px;
  background: blue;
  float: right;
}
.box3{
  height: 100px;
  background: green;
  overflow: hidden;
}
</style>

<div class="wrap">
  <div class="box1">left</div>
  <div class="box2">right</div>
  <div class="box3">main</div>
</div>
```

根据 “浮动盒的区域不会和BFC重叠” 特性，两边的元素浮动，中间的元素开启BFC，很容易就实现了两边固定中间自适应布局。

两边元素浮动就脱离了普通文档流，在普通文档流空出来的位置会被box3填满，这个时候box1、box2是叠在box3上面的，当给box3开启了BFC因为“浮动盒的区域不会和BFC重叠特性”box3只占中间剩余位置。

![](https://ftp.bmp.ovh/imgs/2020/10/c0aa531ccc0e95da.png)

## IFC、FFC、GFC

### IFC

如何创建一个IFC布局：

- 行内块元素（元素的 display 为 inline-block）

IFC的特性

- 在行内格式化上下文中，盒(box)一个接一个地水平排列，起点是包含块的顶部
- 水平方向上的 margin，border 和 padding在盒之间得到保留
- 盒在垂直方向上可以以不同的方式对齐

### FFC

如何创建一个FFC布局：

- 弹性元素（display 为 flex 或 inline-flex）

### GFC

如何创建一个GFC布局：

- 网格元素（display 为 grid 或 inline-grid）