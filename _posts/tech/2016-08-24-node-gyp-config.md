---
layout: post
title: node-gyp编译Native模块报错问题
category: 技术
tags: [node] 
keywords: node, node-gyp 
---

在公司开发机上使用node-gyp编译node Native模块时，经常报以下错误:

![错误](http://shp.qpic.cn/zc_large/0/482_1472050172000/)

咨询了维护tnmp的同事，是因为第一次使用node-gyp编译Native模块时需要下载node源码头文件，默认的download地址是https://nodejs.org，而公司的开发机无法连接外网，所以运行node-gyp时会报错。

tnmp的同事提供了两种解决方法，一种是设置环境变量NODEJS_ORG_MIRROR=http://tnpm.oa.com/mirrors/node/，将下载地址指定为公司内部的私有源，这样开发机就能够从私有源下载头文件了；第二种是手动下载node的源码头文件，并部署到~/.node-gyp/目录下，目录结构如下：

![目录结构](http://shp.qpic.cn/zc_large/0/329_1472050277000/)



