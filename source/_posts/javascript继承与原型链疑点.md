title: javascript继承与原型链疑点
date: 2016-04-19 21:31:44
tags:
- JavaScript
- 原型
categories: JavaScript
---


构造函数，原型和实例的关系如下：
1. 每个构造函数都有一个原型对象
2. 每个原型都有一个指向构造函数的指针
3. 每个实例都包含一个指向原型对象的指针
<!--more-->

<h3>原型链注意的地方</h3>
要点一：**每一个实例也有一个constructor属性，默认调用prototype对象的constructor属性**
```
function SuperType(){}

function SubType(){}

SubType.prototype = new SuperType()
 
 instance = new SubType();
```
此时 `SubType.prototype.constructor`指向`SuperType`，所以`SubType`的实例也是指向`SuperType`！！！可是`instance`明明是构造函数`SubType` 生成的。
所以，这里需要特别注意：**如果替换了prototype对象，那下一步必然要为新的prototype对象加上constructor属性，并将这个属性指回原来的构造函数** 

要点二：子类重写父类中已经存在的一个方法，重写这个方法将会屏蔽原来的那个方法；但是父类的实例调用该方法的时候，还会继续调用原来的那个方法
```
function SuperType(){
    this.food = "fish";
}
SuperType.prototype.getFood = function(){
    return this.food;
}
function SubType(){
    this.food = "egg";
}
SubType.prototype = new SuperType();
SubType.prototype.getFood = function(){
    return this.food;
}
var instance1 = new SubType();
alert(instance1.getFood()); //egg
var instance2 = new SuperType();
alert(instance2.getFood());//fish
```
**Tip:** 给原型添加方法一定要放在替换原型语句之后
要点三：通过原型链实现继承时，不能使用对象字面量创建原型。如果使用对象字面量创建原型方法，那原型包含的是一个Object实例，而不是父类的实例了，因而原型链被切断了。
<h3>原型链的问题</h3>
1) **引用类型值的原型属性会被所有实例共享**，所以我们要在构造函数而不是原型对象中定义属性。但是在通过原型来实现继承时，子类的原型实际上是父类的实例，然后**父类实例的属性会变成了子类的原型属性,**当修改这些属性时，所有实例的属性都受到影响，看如下代码：
```
    function SuperType(){
         this.food  = ["fish","egg"];
    }
    
    function SubType(){
    }
    
    SubType.prototype = new SuperType();
    
    var food1 = new SubType();
    
    food1.food.push("fruits");
    var food2 = new SubType();
    alert(food2.food);
```

解决办法：借用构造函数（经典继承或伪造对象）
```
    function SuperType(){
         this.food  = ["fish","egg"];
    }
    function SubType(){
         SuperType.call(this);//继承SuperType
    }
```

我们可以使用call()和apply()方法在新创建的对象执行构造函数，这样子每次创建SubType对象都会初始化代码，每个实例都有自己的属性，互不影响
2) 在创建子类的实例时，要不影响所有对象的实例情况下不能向超类型的构造函数中传递参数。这个同样可以借用构造函数的技术来解决。

使用构造函数的弊端：
1. 在构造函数中定义的函数无法复用
2. 在父类的原型中定义的方法，对子类型是不可见的

根据 ECMAScript 标准，someObject.[[Prototype]] 符号是用于指派 someObject 的原型。这个等同于 JavaScript 的` __proto__`  属性（现已弃用）。从 ECMAScript 6 开始, [[Prototype]] 可以用`Object.getPrototypeOf()`和`Object.setPrototypeOf()`访问器来访问。


如有错误，欢迎指出！！！




