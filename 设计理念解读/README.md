# jQuery对象的理解
## 1.整体架构
jQuery的核心就是从HTML文档中匹配元素对其执行，他和其他的框架最大的区别就是在实例化的时候是没有进行`new Obj`的操作。
例子：
```javascript
var fov = function(){
	//构造函数
}

fov.prototype = {
	name:function(){
		return 'FovSweet'
	},
	age:function(){
		return '这是秘密！'
	}
}

```
当我们在使用的时候，必须要先实例化一次该函数才能使用
```javascript
var a = new fov();	//fov{}_Object
console.log(a.name());	//FovSweet
```
当然，这个是面向对象最常规的玩法，`jQuery`不是这么玩的。
`jQuery`对象的操作是直接$(selector)即可实例，即在实例的时候返回的直接是`jQuery`对象本身，那么我们可以选择直接在构造函数返回该对象本身。那么根据这种想法，我们换种方式来操作。
```javascript
var fov = function(){
	//构造函数
	retrun new fov()
}
```
结果执行后发现，该种方法会无限实例化，死循环中。。。
于是蛋疼的我回过头去看了看`jQuery`的源码。才发现，`jQuery`采用的是**工厂模式**来进行的操作

>作为一个前端小白的我被鄙视了，请无视这段话。

下面展示一下`jQuery`的操作；
```javascript
var fov = function(){
	//构造函数
	retrun fov.prototype.init();
}

fov.prototype = {
	init:function(){
		return this
	}
}
```
我们在页面上可以直接打印 fov()会发现，返回的就是一个fov对象。
在这，有细心的同学发现了，在`prototype`的`init`中，返回的是一个this，为什么不返回的是一个具体的对象。在这里，this只和当前实例的原型有关，这就另一方面也能够使每一个jQuery对象都是一个独立的个体。在javascript中实例this只跟原型有关系。
那么问题来了，`init`的`this`指向的是`fov`类，如果把`init`函数也当做一个构造器，那么内部的`this`该如何处理呢？
来看下面这段代码
```javascript
var fov = function(){
	//构造函数
	return fov.prototype.init()
}

fov.prototype = {
	init:function(){
		this.age = 18;
		return this;
	},
	name:function(){
		return 'FovSweet'
	},
	age:10000
}

console.log(fov().age)	//18
```
这种情况下，我们会发现，在这里的this指向的其实是fov类，不想在这里对每个独立的fov类做个体的操作，那么就必须设计出一个独立的作用域才行。

## 2.jQuery框架分割作用域的处理

如果我们在每次返回的jQuery类都返回一个新的init实例对象来分隔this，以达到避免this相互混淆，我们就有了下面的这个实验。

```javascript
var fov = function(selector , context){
	retrun new fov.prototype.init();
}

fov.prototype = {
	init:function(){
		this.age = 18;
		return this;
	},
	name:'FovSweet',
	age:20
}

console.log(fov().name())	//Uncaught SyntaxError: Unexpected token new
```
这段运行之后，直接抛出错误，`fov()`类表示他自己不知道什么是name()方法！
很明显的，我们在`return new fov`的时候把`init`和`fov`类的`this`分掉了。

## 3.怎么访问jQuery类原型上的属性和方法？
那么重头戏来了，实现既能够做到作用域的隔离，还能使用jQuery原型对象的作用域的关键点
```javascript
// 给init函数jQuery的原型从而提供此类以后的实例化
jQuery.fn.init.prototype = jQuery.fn;
```
基于这个，我们对之前的代码来进行进一步的修改

```javascript
var fov = function(selector, context) {
	return  new fov.prototype.init();
}
fov.prototype = {
    init: function() {
        return this;
    },
    name: function() {
        return this.age
    },
    age: 20
}

fov.prototype.init.prototype = fov.prototype;

console.log(fov().name()) //20
```
可能到这一步，大部分码农和我一样有点懵逼，为哈这么写？
在这里我很没良心的百度了一张网友的图片来解释一下下。。

![Jquery](http://dl.iteye.com/upload/attachment/0073/2601/3fc8106d-6afd-314c-a6bf-a64157145e67.jpg)

> 这里的fn解释下，其实这个fn没有什么特殊意思，只是jQuery.prototype的引用，也就是我们在进行jQuery拓展的时候经常用的$.fn.extend({...})

## 4.关于Jquery的链式调用
首先对于啥叫js的链式调用这块这里不做详解，最直观简单的解释呢。。
就是链式调用的优点是代码简洁易读，减少了多次重复使用同一个变量，Jquery就是其典型代表，容我举个栗子(⊙o⊙)
```javascript
$('#test').css('width','60px').animate().show().fadeOut()
```
> 当然了。。这种做法纯属蛋疼= =~请无视这段话

那这种做法的优势：
1、节约js代码
2、所有返回的都是同一个对象，可以提高代码的效率

通过简单的`return this`的形式来实现链式调用，同时利用工厂模式来将所有的同一个`dom`对象的操作指定于同一个实例。
来看下面这个栗子

```javascript
fov().init().name()

//分解出来就是酱紫的
a = fov();
a.init();
a.name();
```
这三个返回的都是`fov()`这个对象。
所以，针对于此，我们只需在需要链式的方法的地方返回一个`this`就行了，因为返回的当前实例的可以继续访问自己的原型了。

> 然而这个方法比较糟糕的地方就是没有返回值，所以链式调用不适用于任何环境，只能按需使用

针对于`JavaScript`的非阻塞原理，jQuery在后面的1.5版本引入了`Deferred`对象，后面再说这个问题。
感兴趣的朋友可以参考一下[阮一峰的deferred](http://www.ruanyifeng.com/blog/2011/08/a_detailed_explanation_of_jquery_deferred_object.html)讲解