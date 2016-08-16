---
layout: post
title: React - 通过Webpack来配置React的项目实例
description: 
tag: front-end
category: blog
---
Webpack包含所有的静态资源。两大特色：code splitting 和loader。

code splitting是根据页面自动完成生成js文件的，不需要手动处理。

loader处理各种类型的静态文件，并支持串联操作。

Webpack 本身只能处理 JavaScript 模块，如果要处理其他类型的文件，就需要使用 loader 进行转换。

Loader 可以理解为是模块和资源的转换器，它本身是一个函数，接受源文件作为参数，返回转换的结果。这样，我们就可以通过 require 来加载任何类型的模块或文件，比如 CoffeeScript、 JSX、 LESS 或图片。

先来看看 loader 有哪些特性？

- Loader 可以通过管道方式链式调用，每个 loader 可以把资源转换成任意格式并传递给下一个     loader ，但是最后一个 loader 必须返回 JavaScript。
- Loader 可以同步或异步执行。
- Loader 运行在 node.js 环境中，所以可以做任何可能的事情。
- Loader 可以接受参数，以此来传递配置项给 loader。
- Loader 可以通过文件扩展名（或正则表达式）绑定给不同类型的文件。
- Loader 可以通过 npm 发布和安装。

- 除了通过 package.json 的 main 指定，通常的模块也可以导出一个 loader 来使用。
- Loader 可以访问配置。

- 插件可以让 loader 拥有更多特性。
- Loader 可以分发出附加的任意文件。

###1. npm init

 初始化一个package.json 文件 和 并创建一个node_modules的目录。

###2. npm install webpack --save-dev 

本地安装一个webpack的依赖包，并添加到package.json

###3.touch index.html
添加一个index.html 文件

		<!doctype html>
			<html lang="en">
				<head>
    				<meta charset="utf-8">
   					 <title>New React App</title>
				</head>
				<body>
					<section id="react"></section>
					<script src="bundle.js"></script>
				</body>
			</html>


###4.创建一个入口文件entry.js
		
		import React, { Component } from 'react';
		import { render } from 'react-dom'
		
		const HelloWorld = React.createClass({
  			render() {
   		 		return <div>HelloWorld</div>
 		 }
		})

		render(<HelloWorld />, document.getElementById('react'));
 
###5.通过webpack打包 entry.js 到 bundle.js

![不能找到webpack](/images/githubpages/React-webpac-cannot-find.png)

这是因为我们没有全局安装webpack，直接使用webpack时不能找到此命令。

解决方法有两个：
（1）在项目中已经存在webpack.config.js的前提下，可以在package.json中添加
			
	"scripts": {
  	    "build": "webpack --progress --colors"
	}
运行npm run build即运行webpack。

（2）使用局部webpack脚本: node_modules/webpack/bin/webpack.js ./entry.js bundle.js

###10.npm install react react-dom --save-dev

###11. 不能识别jsx代码，安装相应的loader
![jsx-error](/images/githubpages/webpack-react-with-jsx-error.png)

（1）npm installbabel-loader (ex 6to5-loader) or jsx-loader for JS(X) files,  babel-core,  babel-preset-react, babel-preset-es2015

（2）在更目录下添加 .babelrc 文件
		
		{
  			"presets": ["react", "es2015"]
		}
否则不会识别jsx的代码。

(3) 创建webpack的配置文件 - webpack.config.js
		
		module.exports = {
    		entry: [
   			 "./entry.js"
   			 ],
   	        output: {
       		 path: __dirname,
        	 filename: "bundle.js"
    		},
     		module: {
       		 loaders: [
            	{ test: /\.jsx?$/, loaders: ['babel'], exclude: /node_modules/ }
       		 ]
    		}
		};

###6.引入样式loader:css-loader，style-loader
命令： npm install css-loader style-loader --save-dev 
创建css文件：touch style.css
		
		div {
			background-color: green; 
		}


css-loader 读取css文件
style-loader将样式插入到页面里


引入entry.js,  需添加require("!style!css!./style.css");

require(“./style.css") 而不是添加 require("!style!css!./style.css")的两种解决办法：

#### *打包时添加 —module-bind ‘css=style!css’ 

#### *配置文件webpack.config.js

	module: {
        loaders: [
            { test: /\.css$/, loader: "style!css" }
        ]
    }


###7. 添加webpack-dev-server - npm install webpack-dev-server --save-dev
webpack-dev-server，它将在 localhost:8080 启动一个 express 静态资源 web 服务器，并且会以监听模式自动运行 webpack，在浏览器打开 http://localhost:3030/ 或http://localhost:3030/webpack-dev-server/ 可以浏览项目中的页面和编译后的资源输出，并且通过一个 socket.io 服务实时监听它们的变化并自动刷新页面。

在webpack.config.js文件中添加一个entry-  "webpack-dev-server/client?http://localhost:3030"

在package.json 的script中添加：
		
		"start": "webpack-dev-server  --hot --progress --colors --port 3030"

###9. npm install react-hot-loader --save-dev
React Hot Loader is a plugin for Webpack that allows instantaneous live refresh without losing state while editing React components.
即指在编写react代码时，能够让修改的部分自动刷新，不会引起失去react的state。和网页刷新不同。
在配置文件entry中添加"webpack/hot/only-dev-server"

将
"start": "webpack-dev-server --progress —colors --port 3030” 添加 --hot
或者加入plugin
var webpack = require('webpack');
plugins: [ new webpack.HotModuleReplacementPlugin()]



###8.restructure
根据[文章](https://reactjsnews.com/structuring-react-projects)重组项目结构。 





###12. 添加 jsx的内容
require("../css/style.css");

import React, { Component } from 'react';
import { render } from 'react-dom'

const routes = (
      <div>Hello</div>
)

render(routes, document.getElementById('react'));