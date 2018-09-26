title: canvas图片文字粒子化实现
date: 2018-09-26 16:43:39
tags:
- canvas
- 动画
---



之前无意中发现腾讯的前端开发框架 JX,它官网首页的动画，第一次看到确实是惊艳到我了。[点我看官网效果](http://alloyteam.github.io/JX/)。是的，这么有趣的事情，我当然也要实践实践下。 于是我做了个简化版的，效果如下图：
![image](https://s1.ax1x.com/2018/09/26/iMm0aj.gif)

<!--more-->

##  图片文字粒子化

要实现上面的效果，首先需要把图片或者文字绘制成粒子效果？关于绘图的，首先想到强大的 `canvas`。上帝说要画图，于是有了`drawImage()`,上帝说要画字，于是有了 `fillText()`,上帝说要把图片和文字粒子化，于是它把整个画布的数据交给你`getImageData()`，任你画。


```
//绘制图片
 function loadImage(){
    ctx.clearRect(0,0,width,height);
    ctx.save();
    let image = new Image();
    image.src = "./test.jpg";
    image.onload=function(){
        ctx.drawImage(image,width/2 - 400,50,800,500);
    };
    ctx.restore();
  }
  // 绘制文字
  function particleText(text){
    ctx.clearRect(0,0,width,height);
    ctx.save();
    drawText(text||'SELINAYU');
    ctx.restore();
  }
```

上面已经把要画的图片文字画出来，要实现粒子化，需要先获取画布中的像素数据，再把得到的像素重新绘制，可以把每个像素绘制成圆或者矩形，进行粒子化，事实上，我们一般不会对每个像素进行绘制，对每个像素都绘制首先会导致性能不好，其次不合理，因为粒子是有宽度的。所以，获取像素信息，往往会采取一定的间距的像素数据。

`getImageData() `得到一个一维数组，以RGBA的顺序存储，每4个表示一个像素。可以保存像素信息，用于重新绘制。

```
// 获取并保存像素信息

  function getPixels(){
      let pos=0;
      let imageData = ctx.getImageData(0,0,width,height)
      ctx.clearRect(0,0,width,height);
      dots =[];   
      let data=imageData.data;    //RGBA的一维数组数据
      //i列j行
      for(let i=0;i<=width;i+=radius*2){
          for(let j=0;j<=height;j+=radius*2){
              pos=[ j * width + i ] * 4; //i列j行的位置，取得像素位置
              if(data[pos+3]>0){  
                  let color = `rgba(${data[pos]},${data[pos+1]},${data[pos+2]},${data[pos+3]})`;
                  let pixel={
                      x:i, //重新设置每个像素的位置信息
                      y:j, //重新设置每个像素的位置信息
                      radius: radius,
                      fillStyle:color
                  }

                  dots.push(new Dot(i,j,radius,color))
              }

          }
      }
  } 
```
仔细观察，上面获取的像素间距是 `2 * radius ` 。由于现在的粒子是用圆表示，所以取的两个像素之间间距应该刚好是它们的直径。上面已经获得画布上的像素信息，现在需要把图片或者文字粒子化，对保存的每个像素重新绘制成小圆。

```
// dots 为上面保存像素信息的数组

    for(let i = 0;i < dots.length;i++){
        let  dot = dots[i];
        dot.drawCircle(pos.dx,pos.dy,dot.radius,dot.color); //画圆
    }
```

## 加入动画效果

经过的一些系列操作，我们已经可以得到静态的粒子化图片或者文字，可是我们现在还想添加开始图片的效果，那应该怎么做呢？思路大概是先给粒子一个随机位置，然后给粒子一个加速度，方向为粒子的默认位置，然后把加速度分解为 `x` , `y` 方向。

```
  const Dot = function(centerX,centerY,radius,color){

      this.dx = Math.random()*width;  // 给一个随机的初始位置
      this.dy = Math.random()*height;
 
      this.x = centerX;  // 像素的位置为默认位置
      this.y = centerY;
      this.radius = radius;
      this.color = color;
      this.a = 25; // 小球移动的速度
              
      let temp = Math.sqrt(Math.pow((this.x-this.dx),2)  +  Math.pow((this.y - this.dy),2))
      
      this.vx = (this.x - this.dx)/temp;  // x方向的加速度
      this.vy = (this.y - this.dy)/temp;  // y方向的加速度
      this.updateBallPos = () => {      // 每次更新后的位置
         if(Math.abs(this.dx -this.x) > this.a){
            this.dx += this.vx*this.a;
            this.dy += this.vy*this.a;
         }else{
             this.dx = this.x;
             this.dy = this.y;
         }
         return {dx: this.dx,dy: this.dy}
      }

      this.drawCircle = function(x,y,radius,color){
        ctx.fillStyle = color;
        ctx.beginPath();
        ctx.arc(x,y,radius,0,Math.PI *2);
        ctx.closePath();
        ctx.fill();
    }
  }
```

动画效果当然离不开 `window.requestAnimationFrame`(或者`setInterval`)。所以上面的动画效果是每一帧先清除画布，再重新绘制所有粒子。


```
 updateAllBalls = () => {
    window.requestAnimationFrame(updateAllBalls)
    ctx.clearRect(0,0,width,height);
    ctx.save();
    for(let i = 0;i < dots.length;i++){
        let  dot = dots[i];
        let pos = dot.updateBallPos();
        dot.drawCircle(pos.dx,pos.dy,dot.radius,dot.color)
    }
    let isCancel = cancelAnimation();
    isCancel && window.cancelAnimationFrame(animationFrame)  
    ctx.restore();     
 }
```
## 最后
至此，有动画效果的粒子就完成了。可以[戳我看效果](http://selinayu.cc/Code-of-Practice/canvas-particle/index.html)，[源代码点我](https://github.com/SelinaYu/Code-of-Practice/tree/master/canvas-particle)。