---
title: NodeJS异步IO原理浅析及优化方案
date: 2019-11-15 18:34:00
author: techotter
tags:
  - 异步IO
  - 事件循环
  - 函数式编程
  - 异步控制
categories:
  - NodeJS
---

## 异步IO的是与非

异步IO的好处:

1. 前端通过异步IO可以消除UI堵塞。
2. 假设请求资源A的时间为M,请求资源B的时间为N。那么同步的请求耗时为M+N。如果采用异步方式占用时间为Max(M,N)。
3. 随着业务的复杂,会引入分布式系统,时间会线性的增加,M+N+...和Max(M,N…),这会放大同步和异步之间的差异。
4. I/O是昂贵的,分布式I/O是更昂贵的。
5. NodeJS适用于IO密集型不适用CPU密集型。

<!-- more -->

并不是所有都用异步任务好,遵循一个公式:s= (Ws+Wp)/(Ws+Wp/p),Ws表示同步任务,Wp表示异步任务,p表示处理器的数量。

一些底层的知识:
1. CPU时钟周期:1/cpu主频 -> 1s/3.1 GHz
2. 一波复杂的数学公式
3. 操作系统对计算机进行了抽象,将所有输入输出设备抽象为文件。内核在进行文件I/O操作时,通过文件描述符进行管理。应用程序如果需要进行IO需要打开文件描述符,在进行文件和数据的读写。异步IO不带数据直接返回,要获取数据还需要通过文件描述符再次读取。

## Node对异步IO的实现

完美的异步IO应该是应用程序发起非阻塞调用,无需通过遍历或者事件想象等方式轮询。

1. 应用程序先将JS代码经V8转换为机器码。
2. 通过Node.js Bindings层,向操作系统Libuv的事件队列中添加一个任务。 
3. Libuv将事件推送到线程池中执行。
4. 线程池执行完事件,返回数据给Libuv。
5. Libuv将返回结果通过Node.js Bindings返回给V8。
6. V8再将结果返回给应用程序。

Libuv实现了Node.js中的Eventloop,主要有以下几个阶段:

1. Timers:执行setTimeout和setInterval中到期的callback。
2. Pending callbacks:上一轮循环中有少数的I/O callback会被延迟到这一轮的这一阶段执行。
3. Idle, prepare:仅内部使用。 
4. Poll:最为重要的阶段,执行I/O callback,在适当的条件下会阻塞在这个阶段。
5. Check:执行setImmediate的callback。
6. Close callbacks:执行close事件的callback,例如socket.on("close",func)。

## 几个特殊的API

1. SetTimeout和SetInterval线程池不参与。
2. process.nextTick()实现类似SetTimeout(function(){},0);每次调用放入队列中,在下一轮循环中取出。 
3. setImmediate();比process.nextTick()优先级低。
4. Node如何实现一个Sleep?

几个特殊的API执行顺序:

```js
setTimeout(function () {    
    console.log(1); 
}, 0);

setImmediate(function () {    
    console.log(2); 
});

process.nextTick(() => {    
    console.log(3); 
});

new Promise((resovle,reject)=>{    
    console.log(4);    
    resovle(4); 
}).then(function(){    
    console.log(5); 
});

console.log(6);

// 4 6 3 5 1 2 
```

执行流程:
1. 先执行同步,promise里的按同步执行所以先输出4 6。 
2. 执行完同步后执行nextTick,所以输出3。
3. 接下来执行promise.then里的内容,输出5。
4. setTimeout设置为0ms时,setTimeout先执行,所以输出1 2。如果setTimeout设置了延迟时间的话,则先执行setImmediate。
5. setTimeout、setImmediate、process.nextTick不参与eventloop,都有自己的tracker。

实现一个sleep函数:

```js
async function test() {
    console.log('Hello')
    await sleep(1000)
    console.log('world!')
}

function sleep(ms) {
    return new Promise(resolve => setTimeout(resolve, ms))
}

test()
```

## 函数式编程在Node中的应用

1. 高阶函数:可以将函数作为输入或者返回值,形成一种后续传递风格的结果接受方式,而非单一的返回值形式。后续传递风格的程序将函数业务重点从返回值传递到回调函数中。   - app.use(function(){//todo})。
   - var emitter = new events.EventEmitter;   - emitter.on(function(){//todo})
2. 偏函数:指定部分参数产生一个新的定制函数的形式就是偏函数。Node中异步编程非常常见,我们通过哨兵变量会很容易造成业务的混乱。underscore,after变量。

## 常用的Node控制异步API的技术手段

1. Step、wind(提供等待的异步库)、Bigpipe、Q.js 
2. Async、Await
3. Promise/Defferred是一种先执行异步调用,延迟传递的处理方式。Promise是高级接口,事件是低级接口。低级接口可以构建更多复杂的场景,高级接口一旦定义,不太容易变化,不再有低级接口的灵活性,但对于解决问题非常有效。
4. 由于Node基于V8的原因,目前还不支持协程。协程不是进程或线程,其执行过程更类似于子例程,或者说不带返回值的函数调用。一个程序可以包含多个协程,可以对比与一个进程包含多个线程,因而下面我们来比较协程和线程。我们知道多个线程相对独立,有自己的上下文,切换受系统控制;而协程也相对独立,有自己的上下文,但是其切换由自己控制,由当前协程切换到其他协程由当前协程来控制。

通过本文的分析,我们对Node异步IO的实现原理有了一个基本的认识。Node通过事件驱动、非阻塞IO、单线程的方式,实现了高性能的IO处理。同时,Node也提供了多种异步编程的解决方案,如Promise、Async/Await等,让异步编程变得更加优雅和可控。对Node异步IO的深入理解,可以帮助我们更好地利用Node平台进行高性能服务端开发。
