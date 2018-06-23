---
title: Node.js入门:了解JavaScript运行时环境
date: 2018-06-23 14:16:45
author: techotter
tags:
  - Node.js
  - JavaScript
  - 后端开发
categories:
  - 后端开发
---

Node.js是一个基于Chrome V8引擎的JavaScript运行时环境,它允许我们在服务器端运行JavaScript代码。这意味着,即使你不懂任何服务端编程语言,只要会写JavaScript,就可以编写在服务端运行的程序了。本文将为大家介绍Node.js的基本概念和使用方法,以及一些常用的Node.js模块和应用场景。

<!-- more -->

## Node.js是什么

通常,我们写的JavaScript代码都是在浏览器中运行的。实际上,浏览器就是一个JavaScript运行时环境,用于解释执行js代码。但是,浏览器运行的JavaScript代码处在客户端,只能用于编写前端代码。

Node.js的出现,打破了这一限制。它提供了一个在服务端运行JavaScript代码的环境。虽然Node.js本身是用C++开发的,但它内置了Chrome的V8引擎,可以解释执行JavaScript代码。

关于Node.js,官方是这样解释的:

> Node.js是一个基于Chrome V8引擎的JavaScript运行时。
> Node.js使用高效、轻量级的事件驱动、非阻塞I/O模型。
> 它的包生态系统npm,是目前世界上最大的开源库生态系统。

## 如何使用Node.js

使用Node.js非常简单,有2种方式:

1. 直接运行`node`命令,进入Node.js交互式shell环境,然后在其中编写并执行js代码。例如:

   ```bash
   > node
   > console.log("hello,world") 
   hello,world
   undefined
   > 
   ```

   上面案例中,在shell中键入了`console.log('hello,world')`并敲回车。Node便开始执行该代码,并显示刚才记录的信息,同时打印出`undefined`。这是因为每条命令都会返回一个值,而`console.log`没有任何返回,故输出`undefined`。

2. 使用Node.js执行一个JavaScript文件,这是我们平时最常用的方法。例如:

   ```bash
   > node D:\\Node.js\\workspace\\test-helloworld\\logparserv2.js
   2013-08-09T13:50:33.166Z A 2
   2013-08-09T13:51:33.166Z B 1 
   2013-08-09T13:52:33.166Z C 6
   2013-08-09T13:53:33.166Z B 8
   2013-08-09T13:54:33.166Z B 5
   { A: 2, B: 14, C: 6 }
   ```

   输出内容为对应js文件中的console输出信息。

## Node.js应用举例

Node.js是单线程的。所以,Node.js典型的模式是使用异步回调。基本上,你告诉Node.js要做的事,它执行完后便会调用你的函数(回调函数),这对于Web服务器尤其重要。

在现代Web应用访问数据库的过程中特别普遍,当你等待数据库返回结果的过程中,Node可以处理更多请求。与每次连接仅处理一个线程相比,它使你以很小的开销来处理成千上万个并行连接。

Node.js本身已经内置了许多有用的编程模块,可以用于实现一些有用的功能。而且也已经存在许多独立的框架模块,可以方便地实现以前只能使用笨重的服务端语言实现的功能。下面是两个Node.js应用的例子:

### 1. 解析日志文件

```javascript
// 加载filesystem模块,读取文件
var fs = require("fs");

fs.readFile("D:\\Node.js\\workspace\\test-helloworld\\test.log", function(error, logData){
  if(error) throw error;
  
  var text = logData.toString();
  console.log(text);
	
  // 解析文本 
  var results = {};
  // 将文本分割为多行
  var lines = text.split("\n");
	
  lines.forEach(function(line) {
    // 对每一行文本进行解析
    var parts = line.split(" ");
    var letter = parts[1];
    var count = parseInt(parts[2]);
		
    if(!results[letter]) {
      results[letter] = 0;  
    }
		
    results[letter] += count;
  });
	
  console.log(results);
});
```

日志文件格式如下:

```
2013-08-09T13:50:33.166Z A 2  
2013-08-09T13:51:33.166Z B 1
2013-08-09T13:52:33.166Z C 6
2013-08-09T13:53:33.166Z B 8
2013-08-09T13:54:33.166Z B 5
```

### 2. 编写http服务

```javascript
// 加载http模块
var http = require("http");

http.createServer(function(req, resp) {
  resp.writeHead(200, {"Content-Type": "text/plain"});
  resp.end("Hello! I am a Node.js http server.\n");
}).listen(8080);

console.log("server running on 8080\n");
```

## 为什么要用Node.js

Node.js提供了一种全新的编写后端服务的方式,它并不要求你学习新的编程语言,只要熟悉JavaScript就可以编写后端服务。

但需要注意的是,Node.js仅仅是一个JavaScript运行时环境,单纯安装它并不能做什么。要实现一些有实际意义的功能,还需要使用Node.js内置的或者第三方的模块。

## 常用的Node.js模块 

1. `fs`模块:Node.js自带的模块,可用于访问文件系统(注:在浏览器中执行的js代码是不能访问文件系统的)。
2. `http`模块:Node.js自带的模块,用于构建http服务。  
3. Express框架:第三方模块,可使创建网站的过程十分简单,详见http://expressjs.com/
4. Koa:一个web框架,详见http://koajs.com/
5. Fastify:另一个优秀的web框架,详见https://github.com/fastify/fastify

## 写在最后

1. 编写在Node.js中运行的服务程序,熟练掌握JavaScript语言是基础。同时,编写的程序要遵循JavaScript语言的相应规范和约定。例如:编写模块化程序。

   `myparser.js`:
   
   ```javascript
   // 将日志解析独立成一个模块
   var Parser = function() {
   };
    
   // 解析文本
   Parser.prototype.parse = function(text) {
     var results = {};
     
     // 将文本分割为多行  
     var lines = text.split("\n");
       
     lines.forEach(function(line) {
       // 对每一行文本进行解析
       var parts = line.split(" ");
       var letter = parts[1];
       var count = parseInt(parts[2]);
         
       if(!results[letter]) {
         results[letter] = 0;
       }
         
       results[letter] += count;
     });
       
     return results;
   };
    
   // 告诉Node.js输出文件中的构造函数  
   module.exports = Parser;
   ```

   `logparser.js`:
    
   ```javascript
   // 加载自定义的模块
   var Parser = require("./myparser");
   var fs = require("fs");
    
   fs.readFile("D:\\Node.js\\workspace\\test-helloworld\\test.log", function(error, logData) {
     if(error) throw error;
       
     var text = logData.toString();
     console.log(text);
       
     var parser = new Parser();  
     var results = parser.parse(text);
     console.log(results);
   });  
   ```

在Node.js中还存在一个叫做`npm`的包管理工具,这是Node.js默认的,以JavaScript编写的软件包管理系统,可以类比java中的maven。`npm`会随着Node.js一起安装,`npm`模块仓库提供了一个名为`registry`的查询服务,用户可通过本地的`npm`命令下载并安装指定模块。此外用户也可以通过`npm`把自己设计的模块分发到`registry`上面。

`npm`可以管理本地项目所需模块并自动维护依赖情况,也可以管理全局安装的JavaScript工具。如果一个项目中存在`package.json`文件,那么用户可以直接使用`npm install`命令自动安装和维护当前项目所需的所有模块。在`package.json`文件中,开发者可以指定每个依赖项的版本范围,这样既可以保证模块自动更新,又不会因为所需模块功能大幅变化导致项目出现问题。开发者也可以选择将模块固定在某个版本之上。

