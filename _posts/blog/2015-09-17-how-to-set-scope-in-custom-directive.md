---
layout: post
title: 如何在directive正确设置scope的值
description: 自定义directive设置scope上的值后，必须调用scope.$apply()才能更新到view上
category: blog
---
<img class="rich_media_thumb" id="js_cover" style="width: 300px; height: 300px;" src="https://ooo.0o0.ooo/2015/09/24/560405bbc9043.jpeg">

不知道大家有没有尝试过在自定义的directive更新scope上的值。如果有，你肯定遇到过这个坑。
下面我们来看如下这个实例。

######html
<br/>

```
<body ng-app="exercise">
	<div ng-controller="MyController">
		<p>Can you speak Chinese? {{isAbleChinese}}</p>
		<button type="button" value=“Yes” ng-click="selectYes()">Yes</button>
		<button type="button" value=“No” say-no>No</button>
	</div>
</body>

```
######自定义的directive say-no
<br/>

```
angular.module("exercise", [])
.directive("sayNo", function() {
	var clickNo = function(scope, element, attrs) {
		element.on("click", function() {
			scope.isAbleChinese = 'No';
		});
	};
	return clickNo; 
});

```
打开页面，下面显示如下，

![打开页面](https://ooo.0o0.ooo/2015/09/23/5602a0d195492.png)


可以看到页面有两个按钮，点击他们，分别会将“Cane you speak Chinese?” 的答案Yes或No显示出来。其中，Yes是通过内置指令ng-click来设置答案的，而No按钮时通过自定义directive say-no来处理按钮点击事件。

#####首先我们来看以下No按钮-自定义directive

点击No按钮，页面没有任何变化。

![点击No](https://ooo.0o0.ooo/2015/09/23/5602a0d195492.png)

调试发现，scope中的isAbleChinese已经被赋值为No，但页面仍然没有显示出No，isAbleChinese显示结果仍然是空。

这是为什么呢？在directive中的代码后面添加scope.$apply()试试看会怎么样？

```
angular.module("exercise", [])
.directive("sayNo", function() {
	var clickNo = function(scope, element, attrs) {
		element.on("click", function() {
			scope.isAbleChinese = 'No';
			scope.$apply();
		});
	};
	return clickNo; 
});

```

发现isAbleChinese的值被更新No中。

![加上apply在点击No](https://ooo.0o0.ooo/2015/09/23/5602a78de6572.png)

通过扒代码，可以发现scope的$apply()方法调用了$digest(),而在$digest()中会触发AngularJs的事件循环$digest。$digest会调用相应的watchers列表，而每个watcher会检查当前scope中的值是否有更改，如果有变化，监听函数会被触发model上的值会被更新到view中。

######所以说如果想在自定义的directive设置完scope上的value，想要更新到view上，就需要调用scope.$apply() 方法。

#####那我们现在再来看看内置指令。（内置指令就是以ng开头的directive，angularJs中定义的directive）

点击Yes按钮，可以看到scope中的isAbleChinese被赋值Yes，并且显示到页面。

![点击Yes](https://ooo.0o0.ooo/2015/09/23/5602a2d7cda74.png)


大家可以看到Yes按钮是通过内置指令ng-click来设置isAbleChinese。 但是大家有没有想过为什么在内置directive里更新scope上的值可以立即反应到页面上，我们在使用的时候却没有调用$digest()/$apply(）。

######这是因为所有的内置指令都会将方法包裹在scope.$apply()方法中执行。即执行完方法后都会执行scope.$apply()将改变反映到view上。



