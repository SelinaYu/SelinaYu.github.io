title: 发挥BFC的作用
date: 2017-02-13 23:58:25
tags:
- css
categories: css
---


## 什么是 BFC?
**[戳我查看w3c规范中的BFC定义][1]**
简单的来说，BFC 全称为**Block Formatting Context(块级格式化上下文)**。简单可以理解为一个独立的盒子，规定内部的box如何布局和不受外部影响，也不影响外面的元素。其中一个特点是在BFC中，块盒和行盒都会垂直沿着父元素的边框排列(默认左对齐)。
<!--more-->
## 如何形成BFC?
**只要满足下列条件之一：**

- `float` 的值不是 `none`
- `position` 的值为 `absolute` 和 `fixed`
- `display`的值为下列之一： `table-cell,table-caption,inline-block,flex,or inline-flex.`
- `overflow` 的值不为`visible`

## BFC的运用
**1)解决[外边距合并(塌陷)][2]问题**
HTML
```
<div class="box">
  <p>段落1</p>
  <p>段落2</p>
  <p>段落3</p>
</div>
```
CSS如下：
```
.box {
  background-color: red;
}
p {
  margin: 20px 0;
  background-color: green;
}
```
上面的代码就会导致外边距合并，段落之间的距离未20px,并不是40px。我们可以建立一个BFC去阻止外边距合并的发生。
HTML如下：
```
<div class="box">
  <div class ="BFC">
    <p>段落1</p>
  </div>
    <p>段落2</p>
    <p>段落3</p>
</div>
```
CSS则如下：
```
.box {
  background-color: red;
}
p {
  margin: 20px 0;
  background-color: green;
}
.BFC {
  overflow: hidden;//创建BFC
}

```
**2)清除浮动**
HTML
```
<div class="BFC">
  <div class="box"></div>
  <div class="box"></div>
</div>
```
CSS
```
.BFC {
  border: 5px solid #f00;
  width: 300px;
  overflow:hidden;//当未创建BFC时，父元素高度已经塌陷，两个box脱离父元素的包含块
}
.box {
  border: 5px solid blue;
  width:100px;
  height: 100px;
  float: left;
}
```
**3)用于布局**
HTML
```
<div class="aside"></div>
<div class="main"></div>
```
CSS
```
.aside {
  width: 100px;
  height: 150px;
  float: left;
  background: blue;
}
 .main {
  height: 200px;
  background: #f00;
  overflow:hidden;//创建BFC
}
```
未创建BFC之前，这两个盒子重叠在一起，由于BFC不会与浮动盒子叠加，创建BFC之后，会形成两栏布局。
**4）阻止文本换行**
HTML
```
<div class="box">
  <div class="pic">This is an image</div>
  <p>内部的Box会在垂直方向，从顶部开始一个接一个地放置，其中一个特点是在BFC中，块盒和行盒都会垂直沿着父元素的边框排列(默认左对齐)。
  </p>
</div>
```
CSS
```
.box {
  width: 500px; 
  border: 5px solid yellow;
}
.pic {
  width: 100px;
  background-color: red;
  float: left;
  margin: 10px;
}
p {
  background-color: green;
  overflow: hidden; //创建BFC
}
```
我们常常看到文字围绕图片，是因为图片设置了浮动，图片占据 了一定的宽度，文字就自动环绕着图片。我们可以给文字创建一个BFC,就可以阻止文本换行了。

扩展阅读
[Understanding Block Formatting Contexts in CSS][3]
[CSS布局中一个简单的应用BFC的例子][4]


  [1]: https://www.w3.org/TR/CSS2/visuren.html#block-formatting
  [2]: https://developer.mozilla.org/zh-CN/docs/Web/CSS/CSS_Box_Model/Mastering_margin_collapsing
  [3]: https://www.sitepoint.com/understanding-block-formatting-contexts-in-css/
  [4]: http://www.aliued.cn/2012/12/31/css%E5%B8%83%E5%B1%80%E4%B8%AD%E4%B8%80%E4%B8%AA%E7%AE%80%E5%8D%95%E7%9A%84%E5%BA%94%E7%94%A8bfc%E7%9A%84%E4%BE%8B%E5%AD%90.html