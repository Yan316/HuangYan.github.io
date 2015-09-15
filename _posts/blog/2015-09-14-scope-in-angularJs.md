---
layout: post
title: 关于AngularJs的作用域
description: 关于AngularJs作用域的一些东西
category: blog
---
###定义
AngularJs中的作用域(scope)是定义应用的业务逻辑，控制器方法和视图属性的地方。$scope的所有属性都可以自动被视图访问。基于AngularJs动态绑定，视图上的修改会立即更新到$scope,也可以依赖$scope在其发生变化时立刻重新渲染视图。

###作用域的基本功能：

 1. 提供观察者以监控数据模型的变化。
 2. 可以将数据模型的变化通知给整个应用，甚至是系统以外的组件。
 3. 可以进行嵌套，隔离业务功能和数据。
 4. 给表达式提供运算时所需要的执行环境。

###$scope的生命周期可以分为四个不同的阶段。
#####(1) 创建
 在创建一个新的控制器或者指令(directive)时会创建一个新的$scope, 并且在创建新控制器的函数中将作用域传递进去。

下面列举出会创建新的scope的内置指令：
 
* ng-include
* ng-switch
* ng-repeat
* ng-view
* ng-controller
* ng-if

 除了这些内置指令以外，如果想要在自定义的指令中创建新的scope，可通过设置属性scope为true来实现。


 	angular.module('myApp', [])
 	.directive('myDirective', function(){
   		return {
 			scope: true,
 			....
 		}	    
	 });


#####(2) 链接
 当AngularJs开始运行时，所有的$scope都会附加或链接到视图中。这些作用域会注册到AngularJs应用上下文发生变化时需要的运行函数($watch函数)。

#####(3) 更新
  AngularJs的事件循环被称为$digest循环，这个$digest循环由两个小型的循环组组成，分别是evalAsync循环和$watch列表。
  
  一旦有有事件被触发，事件的处理程序就会在指令的上下文中被调用。实际上是包含作用域的$apply()方法中调用指令。此时,$apply()会触发$digest循环。通常情况，$digest循环从顶层的$rootscope开始，会传播到所有的子作用域。每个作用域都会执行自己的脏值检测，每个监控函数都会检查变化，一旦检测到任意变化就会触发指定的回调函数。
  
  对于$watch列表中检测到的任何变化，angularJs都会再次查看这个列表，已确保没有东西被改变。


#####(4) 销毁
 当一个$scope在视图中不再需要时，这个作用域将会清理和销毁。
 
###父与子-父作用域和子作用域
#####（1）原型继承
$rootScope - ng-app元素同$rootScope进行绑定，它是所有$scope对象的最上层。其他的作用域的父级作用域就是$rootScope.

通常情况下，子作用域会原型继承于父作用域。也就是说他们可以访问父级作用域。默认情况下，AngularJs在当前的作用域中无法找到某个方法或者属性时，便会在父作用域中进行查找。如果也找不到对应的方法或者作用域就会顺着父级的作用域一直向上查找，直到抵达$rootScope为止，如果
$rootScope也无法找到，程序会继续运行，但视图无法更新。

但孤立作用域除外。什么是孤立作用域呢？孤立作用域是指在指令directive内部创建的作用域。即在创建directive时，指令设置了scope:{...},而不是scope=true。

```
angular.moudle('app', [])
.directive('myDirective', function() {
	return {
	scope : {}
	 ...
	};
});
```
#####（2）变量的传递-引用还是复制
大家都知道Javascript对象要么是值复制要么是引用复制， 字符串，数字和布尔变量是值复制。而数值，对象和函数则是引用复制。scope是一个javascipt的对象。

######a. 复制
当子scope继承父scope的字段时，如果字段是个字符串则为复制。此时修改父级对象中的字段同时会修改子对象中的值，但是反之则不行。
	
	<div ng-controller="parentController">
		{{ name }}
		<button ng-click="setNameInParent()"/>
		<div ng-controller="childController">
			{{ name }}
			<button ng-click="setNameInChild()"/>
		</div>
	</div>

	angular.module('myApp', [])
	.controller('parentController',function($parentScope)	{
		$parentScope.name = "Jess";
		$parentScope.setNameInParent = function() {
			$parentScope.name = "I'm father";
		};
	});

	angular.module('myApp', [])
	.controller('childController', function($childScope){
		$childcope.setNameInChild = function() {
			$childscope.name = "I'm children";
		};
	});
此时，点击父controller中的按钮时会同时改变两个值，但点击子controller的按钮时，只会更改子controller的值。

######b.引用
如果当子scope继承父scope的某个对象字段时，该字段会通过引用进行共享。此时修改子scope的属性也会修改父scope中的属性。

	<div ng-controller="parentController">
		{{ ob.name }}
		<button ng-click="setNameInParent()"/>
		<div ng-controller="childController">
			{{ ob.name }}
			<button ng-click="setNameInChild()"/>
		</div>
	</div>

	angular.module('myApp', [])
	.controller('parentController',function($parentScope)	{
		$parentScope.ob.name = "Jess";
		$parentScope.setNameInParent = function() {
			$parentScope.ob.name = "I'm father";
		};
	});

	angular.module('myApp', [])
	.controller('childController', function($childScope){
		$childcope.setNameInChild = function() {
			$childscope.ob.name = "I'm children";
		};
	});
	
此时，无论点击那个按钮，值都会同时修改。

#####（3）事件的传播
点击鼠标，页面滚动等等这些事件，浏览器会响应响应的事件。AngularJs应用也可以响应对应的Angular事件。因为作用域是由层次的，所以可以在作用域链条上传递事件。

######a. 向上传递 - emit冒泡事件
想要把事件沿作用域链向上派送，我们可以使用$emit()函数。
$scope.$emit(eventName, args)

######b. 往下广播 - $broadcast广播

$scope.$broadcast(eventName, args)

不管是向上传递还是往下广播，我们都可以通过监听事件$on来监听。
$scope.$on(eventName, function(evt, next, current) {...});















