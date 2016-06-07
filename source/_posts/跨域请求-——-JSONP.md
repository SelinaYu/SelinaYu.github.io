title: 跨域请求 —— JSONP
date: 2016-01-14 02:38:56
tags: 
- $.ajax
- $.get
- $.getJSON
- 跨域
categories: JavaScript
---

最近帮朋友debug的时候遇到以下的报错，查阅了资料后都说这是跨域的问题，然后我就屁颠屁颠的遇上了JSONP。本篇对JSONP作了一些简单的总结。如有错误，欢迎指正！！
如果直接使用ajax访问，会有以下错误
*XMLHttpRequest cannot load http://www.weather.com.cn/data/cityinfo/101010100.html. No 'Access-Control-Allow-Origin' header is present on the requested resource.Origin 'http://127.0.0.1:8080' is therefore not allowed access.*
<!--more-->
![跨域的报错1]( http://7xq2ky.com1.z0.glb.clouddn.com/jsonperror.png)
在JavaScript中，有一个很重要的安全性限制，被称为“Same-Origin Policy”（同源策略）。所谓同源(域)指当A域与B域具有相同的协议、域名、端口时称为同源(域)。同源策略阻止从一个域上加载的脚本获取或操作另一个域上的文档属性。
但是同源策略不阻止动态脚本插入，并且将脚本看作是从提供 Web 页面的域上加载的。这样我们可以通过script标签实现跨域请求，然后在服务端输出JSON数据并执行回调函数，从而解决了跨域的数据请求。这就是JSONP的内容。
<h3>html代码</h3>
```
    <script type="text/javascript">
    function jsonpCallback(result) {
        console.log(result.data);
    }
    </script> 
    <script type="text/javascript" src="http://www.weather.com.cn/data/cityinfo/101010100.html?callback=jsonpCallback"></script>  
```
注意返回的格式应该是`jsonpcallback({"data":"数据"})`，否则会如下报错![跨域的报错2](http://7xq2ky.com1.z0.glb.clouddn.com/jsonperror2.png)
  
 也许你会问为什么呢？因为用script标签加载完后，会立即把响应当js去执行，很明显`{"data":"数据"}`不是合法的js语句。同时，由于服务器不知道客户端的回调是什么，所以我们传递一个QueryString给服务端，让客户端告诉服务端，回调方法是什么。细心的你可能会发现上面的路径是多了一个`callback`当作参数的，`callback`正是这个QueryString。

<h3>当使用jQuery来实现的话，又是怎样的呢？</h3>

<h4>$.getJSON</h4>
```
    <script type="text/javascript" src="jquery.js"></script>
    <script type="text/javascript">
    $.getJSON("http://www.weather.com.cn/data/cityinfo/101010100.html?callback=?",
        function(result) {
            console.log(result.data);
        });
    </script>

```
<h4>$.ajax</h4>
```
    <script type="text/javascript" src="jquery.js"></script>
    <script type="text/javascript">
    $.ajax({
        url: "http://www.weather.com.cn/data/cityinfo/101010100.html",
        dataType: 'jsonp',
        data: '',
        jsonp: 'callback',
        success: function(result) {
            console.log(result.data);
        },
        timeout: 3000
    });
    </script>
```

<h4>$.get</h4>
```
    <script type="text/javascript" src="jquery.js"></script>
    <script type="text/javascript">
    $.get('http://www.weather.com.cn/data/cityinfo/101010100.html?callback=?', function(result) {
        console.log(result.data);
    }, 'jsonp');
    </script>
```
看了上面三个例子是不是觉得ajax和jsonp这两种技术在调用方式上“看起来”很像，目的也一样，都是请求一个url，然后把服务器返回的数据进行处理。事实上，ajax和jsonp本质上是不同的东西。ajax的核心是通过XmlHttpRequest获取非本页内容，而jsonp的核心则是动态添加`<script>`标签来调用服务器提供的js脚本。

<h3>JSONP的原理</h3>
首先，在客户端注册一个callback函数，然后把callback的名字传给服务器。
接着，服务器生成json数据，然后以callback的名字为函数名动态生成一个function，json数据以入参的方式放置在function中，并返回给客户端。
最后，客户端解析script标签，json数据作为参数，传到预先定义好的函数中。


**需要注意的是**，它只支持GET请求而不支持POST等其它类型的HTTP请求；使用jsonp会在一定程度上造成安全性问题,而且，它只支持跨域HTTP请求这种情况，不能解决不同域的两个页面之间如何进行JavaScript调用的问题。