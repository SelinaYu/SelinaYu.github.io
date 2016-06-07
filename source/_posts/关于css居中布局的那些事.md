title: 关于css居中布局的那些事
date: 2015-12-23 01:16:10
tags:
- css
- 水平垂直居中布局
categories: css
---
平时弄css居中布局的时候，发现有时候有些元素不是不起作用，就和想象的不太一样，所以总结下css的居中布局！！
<!--more-->
<h2>水平居中</h2>
1 **文本水平居中**
`text-align: center`只对行内元素起作用，块状元素无效
2 **使用margin**
```
.child{
  width:200px;
  margin:0 auto;
}
```
缺点：需要指定宽度，当浏览器窗口比元素的宽度还要窄时，浏览器会显示一个水平滚动条来容纳页面,可以采用`max-width:200px;`改善。
3 **使用`table`布局**
```
.child {
  display:table;
  margin:0 auto;
}
```
只需对自身进行设置。
4 **使用绝对定位实现**
```
.parent {
  position: relative;
}
.child {
   position: absolute;
   left:50%;
   transform:translate(-50%);
}
```
兼容性差,IE9及以上可用
5 **使用相对定位**
  
```
.child{
  width:50%;
  position:relative;
  left:50%;
  transform:translate(-50%);
}
```
兼容性差
6 **使用浮动和相对定位来进行水平居中**
和上面相对定位的原理有点类似，把浮动元素相对定位到父元素宽度50%的地方，再把里面的子元素相对定位把多出的一半宽度拉回来。
css的样式
```
    .parent {
        width: 300px;
        height: 200px;
        border: 1px solid red;
    }
    
    .wrap {
        float: left;
        position: relative;
        left: 50%;
        clear: both;
    }
    
    .child {
        border: 1px solid blue;
        position: relative;
        left: -50%;
        white-space: nowrap;
    }
```
html:
```
<div class="parent">
        <div class="wrap">
            <div class="child">水平居中</div>
        </div>
    </div>
```
该布局的优点是不需要知道其宽度，缺点是多了一个包裹层。
7 **使用flex布局**
1)
```
.parent {
  display:flex;
  justify-content:center;
}
```
2)
```
.parent {
  display:flex;
}
.child {
  margin: 0 auto;
}
```
如果进行大面积的布局可能会影响效率

<h2>垂直居中</h2>
1**单行的文字垂直居中**
把文字的`line-height`设为文字父容器的高度，适用于**只有一行文字**的情况。
2**vertical-align**
然而使用`vertical-align`的时候，它必须在`inline-block`和`inline`水平，它才会起作用。
1)
```
.child {
        display: table-cell;
        vertical-align: middle;
        height: 200px;
    }
```
2)
```
    .child {
        display: inline-block;
        vertical-align: middle;
        line-height: 200px;
    }
```
在使用`vertical-align`的时候，由于对齐的基线是用**行高的基线**作为标记，故需要设置`line-height`或设置`display:table-cell`;
3**使用绝对定位**
```
.parent{
  position:relative;
}
.child{
  position:absolute;
  top:50%;
  transform:translate(0,-50%);
}
```
和水平居中的绝对相对定位相似
4**使用flex布局**
```
.parent { 
  display:flex;
  align-items:center;
}
```
<h2>水平垂直都居中</h2>
1 **利用vertical-align,text-align,inline-block实现**
```
.parent {
  display:table-cell;
  vertical-align:middle;
  text-align:center;
}
.child {
  display:inline-block;
}
```

2 **使用绝对定位**
```
.parent{
  position:relative;
}
.child{
  position:absolute;
  top:50%;
  left:50%;
  transform:translate(-50%,-50%);
}
```
3 **另外一种绝对定位居中的方法**
适用于已经知道宽高的元素，若只知道宽只能**水平居中**，只知道高只能**垂直居中**
```
    .child{
        margin:auto;
        position: absolute;
        top: 0;
        bottom: 0;
        left: 0;
        right: 0;
        width: 100px;
        height: 100px;
    }
```
设置了`top: 0; left: 0; bottom: 0; right: 0;` 样式的块元素会让浏览器为它包裹一层新的盒子，因此这个元素会填满它相对父元素的内部空间，这个相对父元素可以是是body标签，或者是一个设置了 `position: relative;` 样式的容器。 
4  **使用display:table-cell来居中**

```
.parent {
        display: table-cell;
        vertical-align: middle;
        text-align: center;
        width: 200px;
        height: 200px;
        border: 1px solid red;
    }
    
    .child {
        width: 50px;
        height: 50px;
        background: black;
        display: inline-block;
    }
```
通过`display:table-cell`来把它模拟成一个表格单元格，这样就可以利用表格那很方便的居中特性`text-align:center`和`vertical-align:middle;`
5 **使用flex布局**
```
.parent {
  display:flex;
  justify-content:center;
  align-items:center;
}
```
如有错误，欢迎指正！！
