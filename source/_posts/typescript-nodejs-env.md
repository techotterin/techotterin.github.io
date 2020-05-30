---
title: 搭建基于TypeScript的Nodejs命令行开发环境
date: 2020-05-30 23:11:12
author: techotter
tags:
  - TypeScript
  - Nodejs
  - CLI
categories:
  - 前端开发
---

本文记录了搭建基于TypeScript的Nodejs命令行开发环境的全过程。

## 为何使用TypeScript

首先,对于编写类库或者工具而言,使用TypeScript的最大好处就是其提供了类型机制,可以避免我们犯一些低级错误。

<!-- more -->

其次,配合编辑器(如VS Code),TypeScript能提供强大的代码提示功能,我们不需要记忆很多API的具体使用,在编写代码时编辑器会自动进行提示。比如引入了http之后,输入http.就会提示可以使用的各个方法和属性,并给出详细的说明。

同是微软旗下,VS Code具有非常强大便利的功能,强烈推荐使用VS Code进行TypeScript和Nodejs开发。

最后,使用TypeScript是大势所趋,很多大公司都在推TypeScript,使用TypeScript开发,可以让我们对TS使用更加熟练。

## 初始化工程

建立命令行工具,需要先创建一个npm包。下文将使用npm工具来完成包的初始化和依赖的安装。

首先创建一个文件夹,然后运行初始化命令:

```bash
mkdir ts-node-demo && cd ts-node-demo
npm init
```

控制台会出现一系列提示, 按照需求输入即可,然后一路回车,完成之后输入yes。

```bash
package name: (typescript-cli) 
version: (1.0.0) 
description: a cli in typescript
entry point: (index.js) 
test command: 
git repository: 
keywords: CLI,TypeScript
author: YourName
license: (ISC) MIT
```

初始化之后本地文件夹会出现一个package.json文件。我们的npm包就已经初始化完成了。为了避免误发布,我们在package.json中做一个更改:

```diff
- private: false,
+ private: true,
```

## 初始化Git

在当前目录下运行:

```bash
git init
```

然后在当前目录创建.gitignore文件,指定忽略node_modules文件夹:

```bash
node_modules/
lib/
```

## 引入Node类型

既然是开发Nodejs程序,为了获得合适的类型校验和代码提示,我们需要引入Nodejs的类型文件:

```bash
npm i -D @types/node
```

## 引入typescript

```bash
npm i typescript
```

然后需要初始化tsconfig文件。

```bash
./node_modules/.bin/tsc --init
```

上述命令会在当前文件夹下面创建一个tsconfig文件,用来指导TypeScript进行编译。

在里面有非常多的配置项,并且有非常详细的解释,我们做两个更改来适配我们的项目:

```diff
+ "sourceMap": true,
+ "outDir": "lib",  
```

上述配置指定生成sourceMap文件,并将TypeScript的编译结果输出到./lib文件夹.

然后在与compilerOptions平级的地方增加选项:

```diff
"compilerOptions": {
    ...
},
+ "include": [
+    "src/**/*"  
+ ]
```

这表示我们只会编译src目录下的.ts文件。

## 编写代码

在当前目录下创建src文件夹,并创建index.ts:

```bash
mkdir src && echo "console.log('Your cli is running.');" > src/index.ts  
```

然后运行:

```bash
./node_modules/.bin/tsc 
```

可以发现在文件夹下出现了lib/目录,里面就是index.ts编译之后的js文件。

## 创建运行脚本

每次编译都需要引用node_modules里面的tsc命令,有些繁琐,有三种方法可以解决:

1. 全局安装typescript包:

```bash
npm i typescript -g
```
就可以直接使用tsc命令了。

2. 使用npx执行
npx是npm提供的命令,其会自动下载对应的包并执行.

```bash
npx tsc
```

3. 创建npm脚本
在package.json中的script中增加一行脚本:

```diff
"script": {
+    "build": "tsc"
}  
```

这里我们采用第3种方法,写入脚本后可以执行:

```bash
npm run build
```

也会成功进行编译。 

## 注册命令

开发Nodejs命令行工具,就是提供一个可以直接调用的命令,而不是使用下面这种方式执行文件:

```bash
node lib/index.js
```

我们想要的效果是执行一个命令就能调用我们的js文件。

首先在当前文件夹创建文件bin/node-cli-demo :

```bash
mkdir bin && touch bin/node-cli-demo.js
```

然后在文件中写入以下内容:

```js
#!/usr/bin/env node
require('../lib/index.js');
```

npm包注册的命令需要在package.json中进行声明,增加如下内容:

```diff
{
    "name": "typescript-cli",
    "version": "0.0.1",
+   "bin": {
+       "node-cli-demo": "./bin/node-cli-demo.js"
+   },
}
```

这表示当执行node-cli-demo这个命令时就去执行我们的./bin/node-cli-demo.js文件。

最后在当前目录调用npm link,这条命令会把我们本地注册的命令放到Nodejs安装目录的bin文件夹下。在安装Nodejs时系统将该文件夹添加到命令查找的路径中。所以我们就可以直接使用我们刚刚注册的命令:

```bash
node-cli-demo
// Your cli is running.
```

## 自动监听文件变动

我们希望每次更改了.ts文件之后,不必手动执行npm run build就能看到最新的效果,可以使用typescript的--watch选项,在package.json中的script中增加start命令:

```diff
{
    "script": {
+       "start": "tsc --watch"
    }
}
```

在当前目录下运行命令:

```bash  
npm start
```

然后对src/index.ts文件做一些更改,另开一个控制台窗口,运行node-cli-demo,会发现打印的内容已经更新了。

这样我们在开发时就只需要关注代码编写,而不用考虑编译的问题了。

接下来我们就可以在src文件里面写我们的具体代码了。
