title: HTML5 实现图片压缩进行上传
date: 2018-09-18 15:28:57
tags:
- canvas
- FileReader
- HTML5
---

##  前言
遇到上传图片，往往会需要对文件过大的图片进行压缩。因为不仅可以节省流量，还可以提升上传速度，提高用户体验。本文主要讲述的是如何利用canvas 对要上传的图片进行压缩上传。

大致的流程如下： 


1) 通过FileReader,获取上传的图片
2) 使用canvas对图片进行drawImage
3) 对画出的图片 toDataUrl 或者 toBlob ,进行质量压缩
4) 上传图片


<!--more-->

## FileReader 获取图片
为什么要使用`FileReader`获取图片，因为`FileReader`web允许应用程序异步读取存储在用户计算机上的文件, 它的实例对象有三种读取文件的方法：
1) `readAsDataURL`: 将文件读取成 `dataUrl` 的形式
2) `readAsBinaryString`: 将文件读取为二进制码
3) `readAsText`: 将文件读取为文本
4) `readAsArrayBuffer` 将文件读取为 `ArrayBuffer` 对象。

Data URL是一种将小文件直接嵌入文档的方案，这里用到的正是 `readAsDataURL`。

## drawImage 画图,尺寸压缩

使用canvas 画图，可以通过定义一个最大宽度，高度，当图片的宽高超出的这个最大的宽高，可以生成一个压缩比，从而计算出画布的大小，控制画出来图片的大小。

## toDataURL, toBlob 质量压缩

`toDataURL`方法返回一个包含图片展示的 data URI 。有两个参数：


```
type: 指定其图片的类型，如: image/png
encoderOptions: 可以从 0 到 1 的区间内选择图片的质量。如果超出取值范围，将会使用默认值 0.92.
```
`toBlob`方法转成blob对象，`toBlob(callback, type, encoderOptions)`三个参数和toDataURL的差不多。

## 获取上传的图片进行画图，压缩上传

```
// HTML 代码
<input id="file" type="file">

// JS 代码
let reader = new FileReader;
let $file = document.querySelector('#file');
let img = new Image();
// 获取图片
$file.addEventListener('change', function (event) {
    file = event.target.files[0];
    if (file.type.indexOf("image") == 0) {
        reader.readAsDataURL(file);    
    }
});
reader.onload = function(e) {  // 异步读取文件
    img.src = e.target.result;
};
// 图片加载成功后，画图，图片大小按比例缩放
img.onload = function(){
    let config = {
        width: 500,
        height: 300,
        level: 0.8   //质量压缩比
    }
  let imgW = img.width;
  let imgH = img.height;
  let resizeW = imgW;  // 图片尺寸按比例缩放后的大小
  let resizeH = imgH;
  let canvas = document.createElement('canvas');
  let context = canvas.getContext('2d');
  
  if (imgW > maxSize.width || imgH > maxSize.height) {
    let multiple = Math.max(imgW / maxSize.width, imgH / maxSize.height);  // 压缩比
    resizeW = imgW / multiple;
    resizeH = imgH / multiple;
  } else {
    return img;
  }
  canvas.width = resizeW;
  canvas.height = resizeH;
  context.drawImage(img, 0, 0, resizeW, resizeH);  // 画图，尺寸压缩
 // let dataURL = canvas.toDataURL('image/png', config.level);  // 质量压缩，这里转成dataURL
 // canvas转为blob并上传
  canvas.toBlob(function (blob) {
    let xhr = new XMLHttpRequest();
    let fd = new FormData();
    fd.append('file',blob);
    xhr.onreadystatechange = function() {
    };
    xhr.open("POST", url );
    xhr.send(fd);    
    }, file.type || 'image/png',config.level);
}
```

## File对象，Blob对象，dataURL 的相互转换


`Data URI` 的格式十分简单，如下所示：

```
data:[<mime type>][;charset=<charset>][;base64],<encoded data>
```
`data`是声明
`mine type` 诸如 `text/html`，`image/png`等
`charset`编码设置，默认编码是 `charset=US-ASCII`还有其他设置`UTF-8`,`base64`等等
`;base64`编码设定
`encoded data`: `Data URI`的实质内容，可以是文本内容，也可以是`base64`编码的内容。

上面说到转换成dataURL的格式，举个栗子如下：

```
data:image/png;base64,iVBORw0KGgoAAAANSUhEUgAABvgAAAG1CAYAAAA81fN...
```

`ArrayBuffer`是JS用于处理二进制数据的一个固定长度的字节序列，这个序列是连续内存的，`ArrayBuffer`只能定义长度，无法直接读取它的内容和直接进行操作。但是，`TypedArray`可以实现对`ArrayBuffer`的操作,`Uint8Array`就是`TypedArray`中的一种。

现在我们了解了`dataURL`的结构 和 `ArrayBuffer`，那么就可以实现 `dataURL`转换成 `File` 对象 和 `Blob` 对象。方法如下： 

### `dataURL`转换成File对象

```
dataURLtoFile = (dataurl, filename) => {
  let arr = dataurl.split(',');
  let mime = arr[0].match(/:(.*?);/)[1];
  let bstr = atob(arr[1]);
  let n = bstr.length;
  let u8arr = new Uint8Array(n);
  while (n--) {
    u8arr[n] = bstr.charCodeAt(n);
  }
  return new File([ u8arr ], filename, { type: mime });
};

```

### `dataURL`转换成Blob对象

```
 dataURLtoBlob = (dataurl) => {
  let arr = dataurl.split(',');
  let mime = arr[0].match(/:(.*?);/)[1];
  let bstr = atob(arr[1]);
  let n = bstr.length;
  let u8arr = new Uint8Array(n);
  while (n--) {
    u8arr[n] = bstr.charCodeAt(n);
  }
  return new Blob([ u8arr ], { type: mime });
};
```

### `File` 对象, `Blob` 对象转换成 `dataURL`

`File` 对象继承 `Blob` 对象，两者的处理其实是一样，转换的方法上面也提到过，使用 `FileReader` 的实例方法 `readAsDataURL`。

```
readBlobAsDataURL = (blob) => {
    let reader = New FileReader();
    reader.onload = function(e){ 
        let result = e.target.result;
        //其他操作...
    }
    reader.readAsDataURL(blob);    
}

```



## 扩展阅读

[JS中的二进制操作简介](http://web.jobbole.com/83701/)
