title: Use CSS Variables
date: 2016-11-30 23:36:54
tags:
- css
categories: css
---

在解决大型站点的时候，常常会面对可维护性的挑战。所用的CSS的往往会出现重复。我们往往会想到使用Less和Sass这些CSS预处理器来定义一个变量，来减少重复的代码。但是，预处理器定义的变量有一个主要的缺点，他们是静态的，不能在运行时修改，而这里要说的CSS variable 正是拥有了这种能力。
<!--more-->
<h3>CSS Variable(CSS自定义属性)的用法：</h3>

- 自定义一个属性：使用格式类似`--name`的名字声明变量； -
- 使用变量：使用`var()`函数,自定义属性支持继承。
```
:root {
	--main-color:red;
}
h1{
	color:var(--main-color);
}
```
<h3>用 JavaScript 操纵它们</h3>
获取属性值的方法`getPropertyValue()`和`getComputedStyle()`,设置属性值的方法`setProperty()`,看下面的代码
```
let h1 =document.getElementsByTagName('h1')[0];
let value = getComputedStyle(h1).getPropertyValue('--main-color');
    h1.style.setProperty('--main-color','green')
```
<h4>静态的预处理器的变量</h4>
文章开始就提到了预处理器的最大不足在于它是静态的，不能在运行时改变。最常见的例子就是媒体查询：
```
$color:red;
@media(min-width:30em){
	$color:green;
}
h1{
	color: $color
}
```
编译出来的结果是
```
h1 { color: red; }
```
正如你所见，它自动忽略了媒体查询的内容。由于你不能根据`@media`改变变量的值，你往往需要对每个媒体查询定义一个唯一的变量，并对每个变量进行唯一的编码。
<h4>预处理器的变量不是级联的</h4>
```
$fontSize:1em;
h1{
	$fontSize:1.5em;
}
body{
	font-size:$fontSize
}
```
上面的代码，`h1`的效果和`@media`一样，会被忽略。编译成
```
body{ font-size:1em;}
```
预处理器的变量还有不支持继承，和第三方样式不可互相操作等各种问题。而CSS Variable则能解决上面提到的预处理器变量的不足。比如css自定义属性能在媒体查询`@media`中重置变量并且把这些变量级联到任何地方去使用
```
:root {
	--main-color:red;
}
@media(min-width:30em){
 :root {
	--main-color:green;
  }
}
h1{
	color:var(--main-color);
}
```




扩展阅读：
[What is the difference between CSS variables and preprocessor variables?][1]
[Why I'm Excited About Native CSS Variables][2]


  [1]: https://css-tricks.com/difference-between-types-of-css-variables/
  [2]: https://philipwalton.com/articles/why-im-excited-about-native-css-variables/