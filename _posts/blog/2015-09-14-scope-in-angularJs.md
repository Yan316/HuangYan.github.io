---
layout: post
title: 关于AngularJs的作用域
description: 关于AngularJs作用域的一些东西
category: blog
---

AngularJs中的作用域(scope)是定义应用的业务逻辑，控制器方法和视图属性的地方。$scope的所有属性都可以自动被视图访问。基于AngularJs动态绑定，视图上的修改会立即更新到$scope,也可以依赖$scope在其发生变化时立刻重新渲染视图。

#####作用域的基本功能：

 1. 提供观察者以监控数据模型的变化。
 2. 可以将数据模型的变化通知给整个应用，甚至是系统以外的组件。
 3. 可以进行嵌套，蛤蜊业务功能和数据。
 4. 给表达式提供运算时所需要的执行环境。

$scope的生命周期可以分为四个不同的阶段。
#####创建
 在创建一个新的控制器或者指令(directive)时会创建一个新的$scope, 并且在创建新控制器的函数中将作用域传递进去。
 $rootScope - ng-app元素同$rootScope进行绑定，它是所有$scope对象的最上层。
 除了$rootScope以外,下面列举出会创建scope的内置指令：
 
* ng-include
* ng-switch
* ng-repeat
* ng-view
* ng-controller
* ng-if

 除了这些内置指令以外，如果想要在自定义的指令中创建新的scope，可通过设置属性scope为true来实现。

```
 angular.module('myApp', [])
 .directive('myDirective', function(){
   return {
 	scope: true,
 }	    .......
 });
```

#####链接
 当AngularJs开始运行时，所有的$scope都会附加或链接到视图中。这些作用域会注册到AngularJs应用上下文发生变化时需要的运行函数($watch函数)。
 每个scope都有自己的watch列表

#####更新
 当事件循环运行时，它通常运行在顶层的$scope- $rootscope上。每个作用域都会执行自己的脏值检测，每个监控函数都会检查变化，一旦检测到任意变化就会触发指定的回调函数。

#####销毁
 当一个$scope在视图中不再需要时，这个作用域将会清理和销毁。









