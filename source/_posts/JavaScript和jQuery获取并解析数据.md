title: JavaScript和jQuery获取并解析数据
date: 2015-11-24 04:53:42
tags:
- Ajax
- JavaScript
- JSON
categories: JavaScript
---
感觉做项目经常会用到它，所以觉得学会ajax发送请求，接收以及如何解析数据很重要。
<!--more-->
JSON是一种轻量级的数据交换语言，是Javascript的一个子集,并且易于让人阅读。要解析JSON数据，首先我们知道可以用HTTP获取数据。所以我们可以创建一个真正的HTTP请求，然后让浏览器代表我们发出请求，做出请求后，浏览器再为我们传回他接受到的数据。

**使用JavaScript获取JSON数据**
```
  var url ="···";                                   //要获取的数据的路径
  
  var xhr = new XMLHttpRequest();                  //创建一个请求对象
  
  xhr.open("GET",url);          //注意这里是用url建立请求，还没打开链接获取数据。第三个参数，是否是异步
  
  xhr.send(null);                            //要发送请求，必须用send(),参数是要发送给服务器的数据
  
  if(xhr.status >= 200)    //检查status属性，确定响应已成功返回
  {
  alert(xhr.responseText); //返回的文本
  }else{
  alert("Request was unsuccessful:" + xhr.status);
  }
```
上面是同步请求，但我们为了让JavaScript继续执行而不必等待响应，常发送异步请求，并且必须在调用`open()`前指定`onreadystatechange`事件处理程序才能确保跨浏览器兼容性。
假设获取以下JSON数据
```
{
    "quiz": [{
        "question": "喝酸奶会不会蛀牙？",
        "choices1": "会", 
        "choices2": "不会"
    }, {
        "question": "下列哪个选项不是复姓的？",
        "choices1": "东理",
        "choices2": "百里"
    }, {
        "question": "哪种睡姿可以预防老年痴呆？",
        "choices1": "侧卧", 
        "choices2": "仰卧"
    }
    ]
}
```
可以参考下面完整的例子，
```
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <title>获取JSON数据</title>
    <meta name="description" content="">
    <meta name="keywords" content="">
    <link href="" rel="stylesheet">
<script src="A/jquery-2.1.4.min.js"></script>
</head>

<body>
    <div id="showjson"></div>
     <script>
   window.onload = function() {
        var url = "quiz.json";
        var xhr = new XMLHttpRequest();
        xhr.onreadystatechange = function() { //看到一些书上用xhr.onload()代替
          if(xhr.readyState == 4){
            if (xhr.status >= 200 && xhr.status <= 304) {
                document.getElementById("showjson").innerHTML = xhr.responseText;
            } else {
                alert("Request was unsuccessful:" + xhr.status);
            }
          }
        };
        xhr.open("GET", url);
        xhr.send(null);

    };
    </script>
</body>

</html>
```
**使用jQuery获取JSON数据**
获取JSON数据，在jQuery中有一个简单的方法 $.getJSON() 可以实现。
`$.getJSON(url,data,success(data,status,xhr))`
该函数是简写的 Ajax 函数，等价于：

```
$.ajax({
  url: url,
  data: data,
  success: callback,
  dataType: json
});
```

`$.each()`是用来在**回调函数**中解析JSON数据的方法。$.each()方法接受两个参数，第一个是需要遍历的对象集合（JSON对象集合），第二个是用来遍历的方法，这个方法又接受两个参数，第一个是遍历的index，第二个是当前遍历的值。
```
<!DOCTYPE html>
<html>

<head>
    <meta charset="utf-8">
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <title>获取JSON数据</title>
    <meta name="description" content="">
    <meta name="keywords" content="">
    <link href="" rel="stylesheet">
<script src="A/jquery-2.1.4.min.js"></script>
</head>
<body>
    <div id="showjson"></div>
     <script>
     $.getJSON("quiz.json", function(data) {
        $.each(data.quiz, function(i, item) {
            $("#showjson").append(
                    "<div>"+item.question+"</div>"+
                    "<ul><li>"+item.choices1+
                    "</li><li>"+item.choices2+"</li></ul>");
        });
        });
  </script>
</body>
</html>
```
遇到较复杂的JSON数据，可能要多使用几个`$.each()`,可见`$.each()`对解析JSON数据挺有帮助的。
**JSON字符串转换为JSON对象**


         var str ='{"name":"Bob","age":14}';
       1. 使用eval()
           var obj = eval('(' + str + ')');
           
       2. 使用`jQuery.parseJSON(str);
       
       3. 使用str.parseJSON();    //会抛出语法异常
       
       4.使用JSON.parse()
            JSON.parse(str);
            
**JSON对象转化为JSON字符串**

      1. 使用toJSONString()
      
      2. 全局方法JSON.stringify()


上面几种方法，除了`eval()`函数是js自带的之外，其他的几个方法都来自json.js包。新版本的 JSON 修改了 API，将 `JSON.stringify() `和 `JSON.parse()` 两个方法都注入到了Javascript的内建对象里面，前者变成了 `Object.toJSONString()`，而后者变成了`String.parseJSON()`。如果提示找不到`toJSONString()`和`parseJSON()`方法，则说明您的json包版本太低。

