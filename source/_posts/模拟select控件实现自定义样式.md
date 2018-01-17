title: 模拟select控件实现自定义样式
date: 2017-04-23 23:30:23
tags:
- css
categories: css
---
最近封装一个分页器的时候需要实现对一个`select`控件自定义样式，然而在我对`option`定义样式的时候，发现他并不起作用。后来在[stackoverflow][1]发现是因为这个元素被OS渲染的，不能通过CSS定义样式。
<!--more-->
如果只是简单的对select样式定义，不涉及option的样式，则可以参考[下面][2]
原理是将浏览器默认的下拉框样式清除，然后应用上自己的。
```
select {
  /*Chrome和Firefox里面的边框是不一样的，所以复写了一下*/
  border: solid 1px #000;
  /*很关键：将默认的select选择框样式清除*/
  appearance:none;
  -moz-appearance:none;
  -webkit-appearance:none;
  /*在选择框的最右侧中间显示小箭头图片*/
  background: url("./arrow.png") no-repeat scroll right center transparent;
  /*为下拉小箭头留出一点位置，避免被文字覆盖*/
  padding-right: 14px;
}
/*清除ie的默认选择框样式清除，隐藏下拉箭头*/
select::-ms-expand { display: none; }
```
由于我写的select控件需要对option定义样式，然后我就自己尝试模拟一个简单的select控件，首先HTML结构如下：
```
<div class="page_select">
 <!-- 模拟select中的值 -->
    <input type="checkbox" id="page_tt" value="10行"/>
    <label for="page_tt" class="page_tt" id="page_tt_label">10行</label>
 <!-- 模拟option选项 -->
    <div class="page_option" id="option">
        <input type="radio" id="radio1" name="radio" value="10行">
        <label class="sizenum" for="radio1">10行</label>
        <input type="radio" id="radio2" name="radio" value="20行">
        <label class="sizenum" for="radio2">20行</label>
        <input type="radio" id="radio3" name="radio" value="50行">
        <label class="sizenum" for="radio3">50行</label>
    </div>
</div>
```
这里主要的原理是`checkbox`，`radio`设置`display:none`,通过它们对应的`label`来控制`checkbox`，`radio`的选中状态来控制`option`选项是否显示和样式的定义。
```
body{
    background: #ccc;
}
.page_select{
    width: 100px;
    height: 25px;
    line-height: 25px;
    position: relative;
    display: inline-block;
}
/*checkbox选中显示option*/
.page_select #page_tt:checked + label + .page_option{
    display: block;
}
 .page_select .page_tt{
     text-align: center;
     background: url("./dropdown.png") no-repeat scroll right center #fff;
     padding-right: 30px;
     border-radius: 12px;
     color: #444444;
     display: block;     
}
.page_select .page_option{
    background: #fff;
    border-radius: 12px;
    overflow: hidden;
    display: none;
    position: absolute;
    width: 100%;
}
/*option的定位*/
.page_select label.sizenum
{
    display: flex;
    flex-direction: column;
    align-items: center;
}
/*option hover的状态*/
 .page_select label.sizenum:hover{
    background-color: #0cbfbe;
    color:#fff;
}

.page_select input[type="radio"],
.page_select input[type="checkbox"] {
    display: none;
}
/*option选中的状态*/
 .page_select  input[type="radio"]:checked + label{
    background: #dddddd;
    color:#444444;
}
```
上面虽然样式可以实现自定义了可是select中的值并没有更新，因为我封装这个分页器的时候是在react中实现的，能直接用他们的state来更新选中的值，在这里，需要JS去更新select中的值：
```
window.onload  = function(){
 let option = document.getElementById("option");
 let page_tt_label = document.getElementById("page_tt_label");
 let page_tt= document.getElementById("page_tt");

    option.onchange = function(e){
       let val = e.target.value;
       page_tt.value = val;
       page_tt_label.innerHTML = val;
    }                           
}
```
[戳我看完整代码和效果][3]


  [1]: http://stackoverflow.com/questions/7208786/how-to-style-the-option-of-a-html-select
  [2]: http://uplifted.net/programming/change-default-select-dropdown-style-just-css/
  [3]: https://codepen.io/selinayu/pen/BRzbpO