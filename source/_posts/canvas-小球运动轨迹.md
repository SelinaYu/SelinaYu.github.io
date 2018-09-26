title: canvas-小球运动轨迹
date: 2018-07-23 20:30:37
tags:
- canvas
- 动画
---


## 前言

canvas中往往有很多动画，其中这些动画往往是通过一些物体的运动轨迹，从而达到一定的视觉效果。本文主要介绍小球在canvas中常见的运动轨迹。比如直线运动，圆周运动，椭圆运动等等。
可以先戳我看效果 [demo](http://selinayu.cc/Code-of-Practice/canvas-ball-path/index.html)

## window.requestAnimationFrame()
canvas的动画效果，一般都是执行一个定时的执行一个方法，这个方法主要包括两步，清除画面，绘制下一步变化的效果。由于残影，我们便可以看到连续的动画效果了。这里我们要提下这个定时器`window.requestAnimationFrame()`,**我为什么不使用`setTimeout,setInterval`?**

<!--more-->

因为`setTimeout,setInterval`不够精确，它们的运行机制决定了时间间隔实际上只是把动画代码添加到浏览器UI线程队列中，并未实际执行。而`window.requestAnimationFrame()`采用系统时间间隔（大部分电脑刷新频率60Hz，每秒重绘60次，动画的最佳循环间隔1000ms/60 ≈ 16.6ms），保持了最佳绘制频率。

## 直线运动

如下图，要想小球，沿着直线运动，是高度不变，x坐标不断变化（x坐标变化的速度就是小球的运动速度），就可以保持在红色水平线上就可以了。要想保证小球匀速运动，x移动的间距必须保持一致，要想小球加速运动，则可以参考下 公式 `x=vt+1/2at^2`。


```
  let uniformX = radius; // x 初始位置 
  let uniformSpeed = 5;
  function uniformRun(){
    animationFrame = window.requestAnimationFrame(uniformRun);
    context.clearRect(0,0,width,height);
    drawLine(0,height/2,width,height/2,'red');  // 画红色线
    drawCircle(uniformX,height/2,radius,'green'); // 画圆，uniformX是 x 的坐标
    if(uniformX < width){
      uniformX += uniformSpeed;
    }else{
      uniformX = 0;
    }
  }
  
  
   // 画实体圆（小球）
  function drawCircle(x,y,radius,color){
    context.save();
    context.fillStyle = color;
    context.globalAlpha=0.95
    context.beginPath();
    context.arc(x,y,radius,0,Math.PI *2);
    context.closePath();
    context.fill();
    context.restore();
  }
```

![canvas-01](https://s1.ax1x.com/2018/09/20/im7Ry9.gif)

## 圆周运动

有着上面直线运动的思路，圆周运动的思路其实也容易想出来，就是同时改变x,y的值，使得小球沿着圆周运动。可是，这里说另一种方法。先把不变的红色圆(运动路径)画出来，然后把 `context`移到圆中心，把运动小球画出来，**然后使用`context.rotate()`不断的旋转画布**，便能把圆画出来了。

```
    function circleRun(){
      animationFrame = window.requestAnimationFrame(circleRun);
      context.clearRect(0,0,width,height);
      context.save();
      drawEmptyCircle(width/2,height/2,circleR,'red'); //画红色圆
      context.translate(width/2,height/2); // 重新映射画布的（0，0）
      context.rotate(Math.PI / 180 * rad );   
      drawCircle(circleX,circleY,radius,'green'); 
      context.restore();
      if(rad < 360){
       rad += angle;     // angle 运动的速度
      }else{
        rad = 0;
      }
    }

```
![canvas-02](https://s1.ax1x.com/2018/09/20/im7TJO.gif)

## 椭圆运动

椭圆运动，用在行星运动轨迹的效果较多。在实现椭圆运动前，先来熟悉下高中的知识，椭圆公式 `x²/a²+y²/b²=1`,三角函数的平方关系 `cos²x+sin²x=1`;椭圆的三角函数表达式可以表示如下`x=acosθ ,y=bsinθ`。椭圆的画法可以把椭圆分割成许多小片段，使用`context.lineTo()`绘制成椭圆。


```
  // 画椭圆，使用lineTo,把椭圆分割许多片段，a,b分别是长轴，短轴
  // 椭圆的三角函数表达式 x = a*cos(t), y = b * sin(t);
  function drawEllipse(color,x,y,a,b){
   //这样可以使得每次循环所绘制的路径（弧线）接近1像素
    let step = (a > b) ? 1 / a : 1 / b;
    context.save();
    context.strokeStyle = color;
    context.beginPath();
    context.moveTo(x + a,y); // 椭圆最右侧开始绘制
    for(let i = 0; i < Math.PI *2; i += step){
      context.lineTo(x + a * Math.cos(i),y + b * Math.sin(i))
    }
    context.closePath();
    context.stroke();
    context.restore();
  }
  
  
  // 椭圆动画的关键代码
  drawEllipse('red',width/2,height/2, ellipseA ,ellipseB ); // 画运动路径椭圆
  drawCircle(width/2 + ellipseA * Math.cos(ellipseTime),height/2 + ellipseB * Math.sin(ellipseTime),radius,'green'); // 小球运动位移
  ellipseTime += ellipseStep; // 控制运动速度

```
![canvas-03](https://s1.ax1x.com/2018/09/20/imHSFf.gif)


## 正弦运动

正弦，余弦运动经常用来制作水波浪效果。继续温习高中知识的正弦 `y = Asin(Bx+C)+D`;


振幅 A： 控制曲线的高度 

周期 (2π/B): 控制宽度

相移 (-C/B): 控制水平移动

垂直位移 D: 控制垂直移动
 
 
 正弦曲线的画法和椭圆有点相似，把正弦曲线分割成很多片段进行绘制。
 
 ```
 // startX,startY 起始位置的x,y轴
   function drawSin(startX,startY,endX,step,A,B,C,color){
    context.save();
    context.strokeStyle = color;    
    context.beginPath();
    context.moveTo(startX,startY);  // 挪到起始位置
    for(let temp = 0;temp < endX;temp+= Math.PI/180 * step ){
      context.lineTo(temp , A * Math.sin(B*temp+C) + startY);
    }
    context.stroke();
    context.restore();
  }
  
  // 关键代码
  
  drawSin(0,height/2,width,sinStep,sinA,sinB,sinC,'red');  
  drawCircle(xSinInit ,height/2 +  sinA * Math.sin(sinB*xSinInit+sinC),radius,'green');
  xSinInit += Math.PI / 180 * sinSpeed;
  if(xSinInit  > width){
    xSinInit = 0;
  }
  
 ```
![canvas-05](https://s1.ax1x.com/2018/09/20/im74dx.gif)

正弦运动往往被用来制作波浪，想象一下，正弦被蓝色的海水填充，然后不断的移动，就能制作简化版的波浪了。如下图(多几条正弦，再好好调节配色，波浪宽度效果更佳)：
![canvas-07](https://s1.ax1x.com/2018/09/20/im7oFK.gif)

## 脉冲运动
先观察下图，可以看出脉冲运动，其实就是把小球放大然后又缩小。一种方法，可以渐变小球的半径，渐变变大达到某个值，又渐变变小。另一种方法可以使用上面提到的三角函数。三角函数的值具有周期性，和脉冲运动的周期特性相似。

![canvas-04](https://s1.ax1x.com/2018/09/20/im7he1.gif)

 

## 绕鼠标旋转

先说这个效果之前，先介绍一下一个三角函数 `Math.atan2(dx,dy)`,dy、dx 两坐标的距离差，可以算出旋转的角度。所以想要鼠标图案绕着鼠标旋转，需要先把画布映射到旋转中心(`context.translate()`),然后计算鼠标和旋转中心的旋转角度，重新绘制图案就可以了。

```
// 监听鼠标事件
 function mouseMoveRun(e){
   let cx = e.clientX;  // 获取鼠标的位置x
   let cy = e.clientY;  // 获取鼠标的位置y
   let dy = cy - height/2;
   let dx = cx - width/2;
   let rotation = Math.atan2(dy,dx);  // 计算鼠标和旋转中心的旋转角度
   context.clearRect(0,0,width,height);
   context.save()
   rotateCube(rotation.toFixed(2),cubeWidth);
   context.restore(); 
 }
 
 // 旋转正方体
 function rotateCube(rotation,cubeWidth){
  context.save();
  context.fillStyle ='green';
  context.translate(width/2,height/2);  
  context.rotate(rotation);  
  context.fillRect(-cubeWidth/2, -cubeWidth/2,cubeWidth,cubeWidth);
  context.fill();  
  context.restore();
}
```
![canvas-06](https://s1.ax1x.com/2018/09/20/im75o6.gif)


##  总结

写完最大的感受，数学很重要！！你想要实现一些更好的效果，就会接触更多的数学知识。路漫漫其修远兮。看源代码，[戳我！](https://github.com/SelinaYu/Code-of-Practice/tree/master/canvas-ball-path)