title: js浅复制与深复制
date: 2015-11-13 20:48:57
tags: js 浅复制与深复制
categories: JavaScript
---
之前听到师兄无意提及的深复制，浅复制，因为不懂所以去了解了一番！
<!--more-->
与其他编程对象一样，JavaScript中对象的拷贝有两种方式：
**浅复制**：并不拷贝对象本身，仅仅是拷贝指向对象的指针。
**深复制**：直接拷贝整个对象内存到另一块内存中。
**demo1**:
``` 
var a = "something";
var a1 = a;
var a = "nothing";
console.log(a1);     //something
```
**demo2**:
```
var a = [0,1,2,3,4];
var a1 = a;
a[0] = "zero";
console.lot(a1); //[zero,1,2,3,4]
```
由上可以得出结论一：

      javascript中基本类型的赋值为深复制，引用类型的赋值为浅复制。

**demo3**:
```
var a ={
  age:20,
  name:"Bob"
  };
  
var a1 = a; 

a = {
  age:18,
  name:"Lucy"
  };
console.log(a1);    //{age: 20, name: "Bob"}
```
由结论一可知，引用类型类型的赋值为浅复制，`a1`应随`a`的变化而变化的。而实际上对象赋值传递的是一个地址，而第8行中`a`的赋值传递了一个新的地址，`a1`依然是原来的地址。
由此可以得出结论二：

      浅复制不会随着存储数据地址的变化而变化，只会随着数据值的变化而变化。

      
js对象可以通过赋值来浅复制，而js对象深复制的方法有：

**使用js提供的方法深拷贝一维数组:**
1.使用`slice`函数

```
var a =[1,2,3,4];
var b =[];

b =a.slice(0);
console.log(b);      //[1, 2, 3, 4]
b [0] = 9;
console.log(a);       //[1, 2, 3, 4]
console.log(b);      //[9, 2, 3, 4]
```
2.使用`concat`函数

```
var a =[1,2,3,4];
var b =[];

//b =a.slice(0);
b = a.concat([]);
console.log(b);
b [0] = 9;
console.log(a);
console.log(b);

```
3.通过`JSON`

`var cloneObj = JSON.parse(JSON.stringify(obj));`

上面这种方法好处是非常简单易用，但是坏处也显而易见，这会抛弃对象的constructor，也就是深复制之后，无论这个对象原本的构造函数是什么，在深复制之后都会变成Object。另外诸如RegExp对象是无法通过这种方式深复制的。
4.通过`jQuery`
`jQuery.extend( [ deep ], target , object1 [, objectN... ] )`
```
var x = {
    a: 1,
    b: { f: { g: 1 } },
    c: [ 1, 2, 3 ]
};

var y = $.extend({}, x),          //shallow copy
    z = $.extend(true, {}, x);    //deep copy

y.b.f === x.b.f       // true
z.b.f === x.b.f       // false
```
`jQuery.extend`注意事项：

    1. 该函数复制的对象属性包括方法在内。此外，还会复制对象继承自原型中的属性(JS内置的对象除外)。
    2. 参数deep的默认值为false，你可以为该参数明确指定true值，但不能明确指定false值。 
    3. 如果只为$.extend()指定了一个参数，则意味着参数target被省略。即将target的每一项合并到jQuery全局对象中。通过这种方式，我们可以为全局对象jQuery添加新的函数。 
    4.如果多个对象具有相同的属性，则后者会覆盖前者的属性值。
    
5。使用`new`一个构造函数进行深复制。
不建议。
会使对象的属性全部暴露在外，遇上私有变量和有参数的构造函数的时候会丢失数据
```
function Man() { 
    var age = 1; 
    this.getAge = function () { 
        return age; 
    } 
    this.grow = function () { 
        age += 1; 
    } 
} 
var man = new Man(); 
```


**尽量避免使用深复制：**

    1. jQuery中深度复制，是将除null,undefined,window对象,dom对象，通过继承创建的对象外的其它对象克隆后保存到target中；之所以排除部分对象，一是考虑性能，二是考虑复杂度（如dom、window对象，如果克隆复制，消耗过大，而通过继承实现的对象，复杂程度不可预知，因此也不进行深度复制）；
    2. 深度与非深度复制区别是，深度复制的对象中如果有复杂属性值（如数组、函数、json对象等），那将会递归属性值的复制，合并后的对象修改属性值不影响原对象
附：下面是jQuery.extend的源码,和jQuery.fn.extend是同一个方法:
```
jQuery.extend = jQuery.fn.extend = function() {
	var src, copyIsArray, copy, name, options, clone,
		target = arguments[0] || {},
		i = 1,
		length = arguments.length,
		deep = false;

	// Handle a deep copy situation
	if ( typeof target === "boolean" ) {
		deep = target;

		// skip the boolean and the target
		target = arguments[ i ] || {};
		i++;
	}

	// Handle case when target is a string or something (possible in deep copy)
	if ( typeof target !== "object" && !jQuery.isFunction(target) ) {
		target = {};
	}

	// extend jQuery itself if only one argument is passed
	if ( i === length ) {
		target = this;
		i--;
	}

	for ( ; i < length; i++ ) {
		// Only deal with non-null/undefined values
		if ( (options = arguments[ i ]) != null ) {
			// Extend the base object
			for ( name in options ) {
				src = target[ name ];
				copy = options[ name ];

				// Prevent never-ending loop
				if ( target === copy ) {
					continue;
				}

				// Recurse if we're merging plain objects or arrays
				if ( deep && copy && ( jQuery.isPlainObject(copy) || (copyIsArray = jQuery.isArray(copy)) ) ) {
					if ( copyIsArray ) {
						copyIsArray = false;
						clone = src && jQuery.isArray(src) ? src : [];

					} else {
						clone = src && jQuery.isPlainObject(src) ? src : {};
					}

					// Never move original objects, clone them
					target[ name ] = jQuery.extend( deep, clone, copy );

				// Don't bring in undefined values
				} else if ( copy !== undefined ) {
					target[ name ] = copy;
				}
			}
		}
	}

	// Return the modified object
	return target;
};
```










