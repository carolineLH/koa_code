# koa源码--基础篇
### Emitter模块和Debug模块
## 前言
在学习koa源码之前你必须是很了解koa，准确的来说是已经阅读过了koa的文档，已经很清楚Koa这个框架的使用。源码学习对初入门koa的同学来说是提升的非常有效的方式。这里我就讲两种koa里面极其重要也是基础的--Emitter和Debug。

### 👉：
感觉对koa不是很清楚的童鞋们，可以戳戳戳！<br>
[koa基础知识文档](http://koa.bootcss.com/) 
[阮一峰的网络日志 koa框架](http://www.ruanyifeng.com/blog/2017/08/koa.html) 
[koa中文文档](https://cnodejs.org/getstart) 

##### 我们还是一步步来吧，从创建文件夹开始，主要对于和我一样的小白们比较好理解。

## 搭建项目：
1. 新建一个文件夹，我的叫koa_code
2. 初始化项目 npm init -y
3. 新建一个文件夹叫app.js
 
### koa的搭建及基础引入：
##### app.js:
```
    const Koa=require('./koa')  //引入koa模块
    const app=new Koa()  //koa是个类，new一个对象app
    // 每个中间件都会获得一个上下文环境
    const main=ctx=>{
        ctx.body='hello world'
    }
    app.use(main) //中间件使用
    app.listen(3001)   //监听3000端口
```
我们这时候就要建一个koa文件夹了，我们现在是要自己写koa哦！

##### koa文件夹下index.js文件：
```
    // 事件触发与事件监听功能的封装
    const Emitter=require('events') //从事件模块events引入Emitter
    const http=require('http')  //引入http模块
    const debug=require('debug')  //引入debug模块
    //向外暴露这个类
    module.exports=class Application extends Emitter{
        //继承于Emitter类，所以子类必须声明constructor类，来自es6语法
        constructor(){
            super()
            // 放中间件的数组
            this.middleware=[]
        }
        // 启用中间件  中间件就是函数
        use(fn){
            console.log(fn)
            //把fn（函数）这个参数放入中间件数组
            this.middleware.push(fn)
            // chain链式调用
            return this
        }
        //listen函数实现
        // 运用...reset，es6语法。里面参数有端口号，回调函数等等
        listen(...args){
            //开始监听
            debug('listen')
            // 将http的server封装一下
            const server=http.createServer(this.callback())
            return server.listen(...args)
        }
        //创建服务server回调函数的实现。
        callback(){
            const handleRequest=(req,res)=>{
                res.end('hello world')
            }
            return handleRequest
        }
    }
```
已经抛出了Emitter和Debug了。在这里实现了监听函数和中间件函数。在代码中每一步都有注释，可以很清楚的理解。

现在访问3001端口，页面上就显示出了：
![](https://user-gold-cdn.xitu.io/2017/9/8/9a0ac1b7b0285bb425af4f7d8850ba89)

## Emitter
koa是基于事件机制的web框架 只是事件起源在用户端，结束在服务器端。<br>
emitter是node.js对事件订阅及发布机制的抽像对象，原生实现了emit方法，可以在框架内部随意添加事件。emitter使得事件一切皆可订阅一切皆可触发。

下面来看看emitter的厉害之处，新建一个文件夹Emitter,建一个文件test.js
##### Emitter/test.js:
```
    const Emitter=require('events')  //引入Node.js事件模块 events
    const EventEmitter=Emitter.EventEmitter;  //引入Emitter模块中的EventEmitter模块
    const event=new EventEmitter()  //new 一个对象event
    // 订阅 就像click.on一样
    event.on('some_event',function(){
        console.log('some_event事件触发');
    })
    setTimeout(function(){
        // 调用 事件执行
        //'some_event要执行的事件
        event.emit('some_event')
    },1000)
```
这里充分说明了Emitter的事件机制，在控制台能够输出：
![](https://github.com/carolineLH/vue_eleme/blob/master/p2.png)
emit与自定义事件有关，我们知道，父组件是使用props传递数据给子组件，但如果子组件要把数据传递回去，应该怎样做？那就是自定义事件！
对emit不太理解的童鞋戳戳戳👉：[vue组件之emit](http://www.jianshu.com/p/2e29f6e8800b) 

## Debug
debug框架中常用的工具,debug提示当前程序执行到哪里.在项目中总是要用到对项目的调试，就要用到debug.

现在开始讲debug模块:

1.新建一个文件夹debug,新建两个文件：test.js work.js
2.yarn add debug
3.新建终端，切到debug,
        npm init -y
        yarn add debug

##### test.js:
```
    //引入debug模块，并且打印debug出处
    // ('http')里面http标签表示打印debug出处
    var debug=require('debug')('http'),
    http=require('http'),  //引入http模块
    name='My App'  //字符串
     // %o指向对象
    debug('booting %o',name) //debug运行，打印
    //创建http请求服务
    http.createServer((req,res)=>{
        //debug打印请求的方法和url
        debug(req.method+''+req.url)
        //请求返回字符串
        res.end('hello\n')
    }).listen(8080,function(){
        debug('listening') //开始监听
    })
    // cross-env跨环境执行环境变量
    require('./work')  //引入文件work.js
```
这时候package.json文件中的"scripts"中设置：
"dev":"cross-env DEBUG=http nodemon test.js"打开控制台，可以看到控制台输出：
![](https://user-gold-cdn.xitu.io/2017/9/8/5aca260c9307a1fb6d133075e1130208)

### cross-env:
cross-env 跨环境设置环境变量。在package.json文件中的"scripts"中设置：
"dev":"cross-env DEBUG=标签 nodemon test.js"
更好的理解cross-env戳戳戳👉:[cross-env解决跨平台设置NODE_ENV问题](https://segmentfault.com/a/1190000005811347?_ea=934705)
##### work.js:
```
    var a = require('debug')('worker:a')
    var b = require('debug')('worker:b')
    // debug函数
    function work() {
        a('doing lots of uninteresting work')
        // 使得能够一直执行
        setTimeout(work, Math.random()*1000)
    }
    work()
    // debug函数
    function workb() {
        b('doing some work');
        // 使它能够一直执行
        setTimeout(workb, Math.random()*2000)
    }
    workb()
```
这时候package.json文件中的"scripts"中设置：
"dev":"cross-env DEBUG=* nodemon test.js"控制台上输出：
![](https://user-gold-cdn.xitu.io/2017/9/8/fac09500df5f99b2a5ba56b620b7eb47)
## 小结：
Koa使用了ES6规范的generator和异步编程是一个更轻量级Web开发的框架，需要深入理解其底层实现，对koa源码的学习要慢慢的深入。

各位大佬们有什么建议欢迎提出哦🙂
