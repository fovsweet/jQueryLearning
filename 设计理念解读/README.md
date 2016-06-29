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
当然，这个是面向对象最常规的玩法，jQuery不是这么玩的。
jQuery对象的操作是直接$(selector)即可实例，即在实例的时候返回的直接是jQuery对象本身，那么我们可以选择直接在构造函数返回该对象本身。那么根据这种想法，我们换种方式来操作。
```javascript
var fov = function(){
	//构造函数
	retrun new fov()
}
```
结果执行后发现，该种方法会无限实例化，死循环中。。。
于是蛋疼的我回过头去看了看jQuery的源码。才发现，jQuery采用的是工厂模式来进行的操作（ps：作为一个前端小白的我被鄙视了，请无视这段话。）
下面展示一下jQuery的操作；
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