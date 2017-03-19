---
title: "从零开始学习React: 使用Yarn, Webpack和Babel来搭建环境"
date: 2017-03-06 16:26:51
tags: [Programming, JavaScript, React]
categories: programming
---

### 序言
最近我开始学习炙手可热的[React](https://facebook.github.io/react/)，在浏览了官网的[Get Started](https://facebook.github.io/react/docs/hello-world.html)部分之后，我想动手写一写代码，这才发现官网并没有给出详细的环境搭建过程，而且我并不想使用官网提供的boilerplate项目[Create React App](https://github.com/facebookincubator/create-react-app)。因此，我把这个过程记录下来，希望可以给新手们一点帮忙，这里并不介绍`React`相关的基础知识，只介绍环境的搭建过程。来，让我们从零开始，跟着贾思敏姐姐一步步搭建一个`Hello Jasmine`的`React`项目。

### 创建项目目录
在安装后`Git`后，为项目新建一个目录，并初始化git。

	mkdir into-react
	cd into-react
	git init
	
	
### 跟着急性子的我先写Hello Jasmine
急性子的我，已经等不及想写点`React`代码了，我怕一步步部署好`webpack`，写好`.babelrc`等等之后，热情就没了，所以让我们先来写这个神奇的`jsx`文件，并命名为`index.jsx`。

	import React from 'react';
	import ReactDOM from 'react-dom';

	ReactDOM.render(
  		<h1>Hello, Jasmine!</h1>,
  		document.getElementById('root')
	);

写完这个文件时候，我们肯定是希望把它放进浏览器里面展示出来，那么可能的问题有以下几种：

+ 还没有`HTML`文件，也还没有`id`为`root`的元素
+ 怎么把`jsx`文件放进`HTML`页面?
+ `import`这个高端的语法没见过?
+ `react`和`react-dom`还没有安装，怎么`import`?

让我们一个个问题来解决。

### 新建HTML文件
首先，让我们新建包含`id`为`root`元素的`index.html`, 内容为：

	<html>
    	<head>
        	<meta charset="utf-8">
        	<title>Into React</title>
    	</head>
    	<body>
        	<div id="root"></div>
    	</body>
		</html>

### 使用Webpack把需要的资源文件打包
接下来的问题就是，`index.jsx`中使用了`react`和`react-dom`，要让`index.jsx`运行起来，我们的页面中需要包含`react`和`react-dom`。我们使用`Webpack`来把所有内容打包到`bundle.js`，然后在`index.html`中添加`bundle.js`。首先，使用`yarn init`初始化项目（可以一路回车完成配置），然后使用`yarn`在项目中安装`Webpack`。然后我们把`node_modules`放进`.gitignore`中。

	yarn init
	yarn add webpack --dev	
	touch webpack.config.js	
	
`webpack.config.js`的内容为：
	
	var BUILD_DIR = path.resolve(__dirname, 'src/client/public');
	var APP_DIR = path.resolve(__dirname, 'src/client/app');

	var config = {
    	entry: APP_DIR + '/index.jsx',
    	output: {
        	path: BUILD_DIR,
        	filename: 'bundle.js'
    	}
	};

	module.exports = config;

我们顺便调整了一下项目目录结构，将所有代码相关文件都放进`src`目录下的`client`文件夹中。其中`app`文件夹放源文件，`public`文件夹里面放打包后的`bundle.js`文件。
	
然后，我们把下面一行添加到`index.html`文件里面：

	<script src="public/bundle.js" type="text/javascript"></script>

有热心网友遇到了以下错误：
<img src="http://omcdckn46.bkt.clouddn.com/react-error.png" width="600px">

原因是把`bundle.js`放在了`head`里面，最好是放在`body`最下面，确保这个时候`DOM`里面已经有`id`为`root`的`div`了。如果一定想放在`head`里面，可以在`script`里面加一个`defer`的attribute。
	
	<script src="public/bundle.js" defer type="text/javascript"></script>

`defer`可以保证该`js`在`HTML`解析完成后，再进行执行。当然了，也可以把`ReactDOM.render`放进`$(document).ready()`里面。`defer`和`async`还有不加的区别可以参考[这篇文章](http://www.growingwiththeweb.com/2014/02/async-vs-defer-attributes.html)。	

当前我们的项目结构如下：

<img src="http://omcdckn46.bkt.clouddn.com/1.png" width="200px">

### 使用Babel转换ES2015语法和JSX
当我们使用`webpack webpack.config.js`来生成`bundle.js`时，我们发现会报错：

	Unexpected token (5:4)
	You may need an appropriate loader to handle this file type.
	
就是说`webpack`不认识`jsx`文件，我们需要对应的`loader`让`webpack`认识它。这时我们就需要`Babel`了。	首先我们安装`babel-loader`，让`babel`来处理所有`jsx`文件。

	yarn add babel-loader --dev

在添加的时候我们发现有个warning：

	warning "babel-loader@6.3.2" has unmet peer dependency "babel-core@^6.0.0".

因此我们还需要安装`babel-core`。

	yarn add babel-core --dev

然后我们就需要挑选合适的`babel`插件来转换我们的`index.jsx`文件。我们仔细来看这个文件，发现首先`import`了`react`和`react-dom`。首先，按照官网[Installation](https://facebook.github.io/react/docs/installation.html)的介绍，我们安装React和React-dom。使用Yarn安装:
	
	yarn add react react-dom	
	
接下来，什么是`import`？`import`其实是ES2015中引进的新语法，ES2015引入了`class`的概念，关于ES2015的简单了解可以参考[<ECMAScript6 入门>](http://es6.ruanyifeng.com/)。那么，为了让各种浏览器能够理解`import`，我们需要将它转换成ECMAScript 5的语法。这里，我们就需要用到`preset-es2015`这个插件。而为了让转化`jsx`语法，我们还需要使用`preset-react`。
	
	yarn add  babel-preset-react babel-preset-es2015 --dev

再修改我们的`webpack.config.js`文件:

	var webpack = require('webpack');
	var path = require('path');

	var BUILD_DIR = path.resolve(__dirname, 'src/client/public');
	var APP_DIR = path.resolve(__dirname, 'src/client/app');

	var config = {
    	entry: APP_DIR + '/index.jsx',
    	output: {
        	path: BUILD_DIR,
        	filename: 'bundle.js'
    	},
    	module: {
        	loaders: [
            	{
                	test: /\.jsx?/,
                	include: APP_DIR,
                	loader: 'babel-loader'
            	}
        	]
   		}
	};

	module.exports = config;


这个时候使用`webpack`来编译项目，发现如下错误：

<img src="http://omcdckn46.bkt.clouddn.com/2.png" width=600px>

我们虽然指定了`babel-loader`来处理`jsx`文件，也安装了转换文件所需要的`babel-es2015`和`babel-react`，但我们在项目里并没有指定用这两个插件。因此，我们在根目录下添加`.babelrc`文件：

	{
    "presets": ["react", "es2015"]
	}
	
`babel-loader`会自动去读这个文件，再次运行`webpack`，我们发现build已经成功了。

### 部署服务器来看我们的Hello！
现在离看到我们的页面只差一步了，就是起一个服务器。我们安装`webpack-dev-server`来启动并`watch`项目，这样当你修改代码的时候，就可以自动编译并显示在浏览器上。

	yarn add webpack-dev-server --dev
	
可以使用

	node_modules/.bin/webpack-dev-server --content-base src/client

来启动服务器。也可以把它作为一个`script`加到`package.json`里面：

	"scripts": {
    	"dev": "node_modules/.bin/webpack-dev-server --content-base src/client"
  	}	
  	
这样只要运行`yarn run dev`，打开`localhost:8080`就能看到页面了！

<img src="http://omcdckn46.bkt.clouddn.com/3.png" width=250px>  	