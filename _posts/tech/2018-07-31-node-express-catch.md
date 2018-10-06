---
layout: post
title: node express框架异常捕获实践
category: 技术
tags: [node, express]
keywords: node, express
---

node是一个比较特殊的后台编程环境，事件驱动异步单线程的处理方式使node具有高性能的特点，但同时也让node变得非常脆弱，异步使得node的异常不好捕获，单线程使得node异常崩溃会导致极为严重的后果，为了应对这个问题，下面介绍一下基于express框架的node异常捕获的现网实践方案。

简单来说就是express错误处理中间件+domain，先说下express错误处理中间件捕获异常的问题，express是一个中间件队列的结构，每一个中间件或注册若干layer，每一个layer注册一个处理任务的handle，express通过try catch包裹handle捕获异常，捕获到异常后交给错误处理中间件处理，但是handle中的异步处理过程发生异常express是无法捕获的!

通过下面代码说明下：

    var express = require('express');
    var app = express();
    var assert = require('assert');

    app.get('/hello', function(req, res, next){
        console.log("enter hello");
        assert(false);  //这里主动触发异常
        res.send("hello");
    });

    app.use(function(err, req, res, next){
        if(err){
            console.error(err);
            res.send("catch err");
        }else{
            next();
        }
    });

    app.listen(8000);

执行命令`curl http://127.0.0.1:8000/hello`，返回结果catch err，异常被捕获了

    var express = require('express');
    var app = express();
    var assert = require('assert');
    
    app.get('/hello', function(req, res, next){
        console.log("enter hello");
        setTimeout(function(){
            assert(false);  //这里主动触发异常
            res.send("hello");
        }, 1000);
    });
    
    app.use(function(err, req, res, next){
        if(err){
            console.error(err);
            res.send("catch err");
        }else{
            next();
        }
    });
    
    app.listen(8000);

程序崩溃没有返回结果，由此可见express无法捕获异步异常。

所以捕获异步异常就需要用到domain模块了，domain的原理是改写了`process._fatalException`函数，使得该函数首先调用domain注册的函数处理异常，代码示例如下:

    var express = require('express');
    var app = express();
    var assert = require('assert');
    var domain = require('domain');
    
    app.use(function(req, res, next){
         var d = domain.create();
         d.add(req);
         d.add(res);
         d.run(function(){
            next();
         });
         d.on('error', function(err){
            console.error(err);
            res.send("domain catch err");
         });
    });
    
    app.get('/hello', function(req, res, next){
        console.log("enter hello");
        setTimeout(function(){
            assert(false);  //这里主动触发异常
            res.send("hello");
        }, 1000);
    });
    
    app.use(function(err, req, res, next){
        if(err){
            console.error(err);
            res.send("catch err");
        }else{
            next();
        }
    });

    app.listen(8000);

执行`curl http://127.0.0.1:8000/hello`返回domain catch err


