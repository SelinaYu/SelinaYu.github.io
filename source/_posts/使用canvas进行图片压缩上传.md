title: 使用canvas进行图片压缩上传
date: 2016-09-11 14:56:09
tags:
- Canvas
- FileReader
categories: html5
---

  最近项目有个需求是要前端进行图片压缩，在这里总结下，整个过程主要涉及到html5的三个API:FileReader,Canvas,FormData,进行了尺寸压缩和质量压缩。
  
主要思路：
1）通过input file上传图片，使用FileReader读取上传的图片数据
2）将图片数据传给img,然后使用canvas绘制图片，调用canvas.toDataURL(),进行压缩
3）获取压缩后的base64格式的图片数据，转换成blob对象
4) 使用formdata进行上传
<!--more-->

**1)将input中选择的图片通过FileReader获取**

```
<input type="file" id="files" name="pic" accept="image/*"/>
```
FileReader读取图片数据
```
var filesDom = document.getElementById("files");
filesDom.onchange = function(){
   var files = filesDom.files[0],
       image1 = new Image(),//原始图片
       image2 = new Image(),//压缩之后的图片
       reader = new FileReader(),
       canvas = document.createElement('canvas'),
       context = canvas.getContext('2d');
       url;
   
       reader.onload = function(e){
          url = e.target.result;
          image.onload = function(){
             /***/
          }
          image.src = url;
       }
       reader.readAsDataURL(files);
}
```
**2)加载完img之后，canvas使用drawImage方法绘制图像(尺寸压缩)，同时使用toDataURL方法可以进行质量压缩**

```
   image.onload = function(){
       var Obj = sizeCompress(image);//这个方法返回的是尺寸压缩的宽和高
       thumbH= Obj.thumbH;
	   thumbW = Obj.thumbW;
	   canvas.width = thumbW;
	   canvas.height = thumbH;
	   
	   context.drawImage(image,0,0,thumbW,thumbH);
	   dataUrl = canvas.toDataURL("image/jpeg");//
      
     blob = dataURLtoBlob(dataUrl);//转换成blob对象
     while(blob.size >200*1024){
     //当图片大小>200KB对图片进行图片压缩和尺寸压缩，了解更多看我的完整代码
     }
   }
```

需要注意的是：
>toDataURL默认格式为image/png,只有指定格式为image/jpeg或image/webp，才可以选择质量压缩的值(即第二个参数)。而且按原宽和高使用image/png还可能让图片变大。

**3) 把base64格式的图片数据转换成blob对象**
```
function dataURLtoBlob(dataURL) {
  var binary = atob(dataURL.split(',')[1]);
  var array = [];
  for(var i = 0; i < binary.length; i++) {
      array.push(binary.charCodeAt(i));
  }
  return new Blob([new Uint8Array(array)], {type: 'image/jpeg'});
}	
```

这里先使用atob进行解码(编码的方法btoa()),然后以创建8位整数格式的arraybuffer，传入blob构造函数创建blob对象。

**4)上传文件**
```
    var fd = new FormData(),
        xhr = new XMLHttpRequest();
    fd.append('file', blob);
    
    xhr.open("POST",url);
    xhr.onreadystatechange = function() {
        //do something
    }
    xhr.send(fd);
```
完整的代码demo：[戳这里][1]

如有错误，欢迎指出！！！

参考链接：
**深入研究html5实现突破压缩上传：** http://www.cnblogs.com/hutuzhu/p/5265023.html
**FileReader(MDN):** https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader
**应用图像Using images：**
https://developer.mozilla.org/zh-CN/docs/Web/API/Canvas_API/Tutorial/Using_images
    
[1]:https://codepen.io/selinayu/pen/oZLyjy

