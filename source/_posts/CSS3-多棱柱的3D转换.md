title: CSS3-多棱柱的3D转换
date: 2018-07-12 15:52:10
tags:
- css3
- 3D
- 动画
---

之前看过相关文章，看到一些3D的效果，往往觉得很惊艳，虽然自己也偶尔用到CSS3的动画，但更多的是使用2D的偏移，对3D更多的认知是停留在知道这个属性，并未进行实践。最近有空，刚好尝试了实现一个正多棱柱生成的动画，戳我看 [demo](http://selinayu.cc/Code-of-Practice/css3-multi-prism/index.html),效果图如下：
![image](http://7xnpna.com1.z0.glb.clouddn.com/css3-multi-prism.png)


<!--more-->

## transform-style


```
transform-style: flat|preserve-3d;
```

默认值是`flat`,不保留子元素的3D位置，表示该元素在所在的平面内被扁平化。`perserve-3d`保留子元素的3D位置。倘若不设置该属性，3D效果是平面的，不会独立存在三维空间里。一般声明在要进行3D转换元素的**父元素上**(舞台元素)。


## perspective

有人理解为透视，也有人理解为景深。简单来说，就是眼睛看到元素的距离。值越大，眼睛看元素的距离越远，看到的元素相对就越小。所以，当透视距离固定时，`translateZ`的越大，与透视点越近，看到物体会更大，范围更小。近大远小，了解一下。`perspective：none/0`时，所有后代元素被压缩在同一个二维平面上，这时就不存在透视的效果。

`perspective`属性有两种书写形式，一种用在舞台元素上（动画元素们的共同父辈元素）；第二种就是用在当前动画元素上，与transform的其他属性写在一起。


## 正方形的3D转换

这里不考虑上下两边。首先需要准备4个div,长宽分别是400px,层叠在一起，然后围绕Y轴(rotateY)，分别旋转90°，180°，270°，360°。效果如下图：

![image](http://7xnpna.com1.z0.glb.clouddn.com/3D-01.jpg)

然后在`translateZ(200px)`,就能制作成一个正方形。
同理，多棱柱的情况也是如此。假设我们要制作 N 棱柱，那么我们每个div需要旋转的间距就是 `360°/N`,生成如下效果
![image](http://7xnpna.com1.z0.glb.clouddn.com/3D-02.jpg)
然后我们重点是要计算`translateZ`的距离，我们观察正多边形，宽度同样是400px，六边形，如下图。观察图可知，`translateZ`的距离其实就是正N边型的圆心到边的距离。
![image](http://7xnpna.com1.z0.glb.clouddn.com/3D-03.jpg)

所以`translateZ`偏移的值可进行如下计算

```
    let deg1 = 360/N; // 旋转间距
    let deg2 = deg1/2; // 用于计算偏移 
    // 弧度＝(角度/180) *PI
    let rad = deg2 * Math.PI / 180;
    let w  = width/2;
    let z;  // Z轴偏移量
    // Math.tan(rad) = w/h;
    z = w / Math.tan(rad);
    
```
所以六边形的Z轴偏移量是

```
z = 200 / Math.tan(30 * Math.PI /180)  ≈  346.41
```

## 最后

[戳我看效果！！效果！！](http://selinayu.cc/Code-of-Practice/css3-multi-prism/index.html)

