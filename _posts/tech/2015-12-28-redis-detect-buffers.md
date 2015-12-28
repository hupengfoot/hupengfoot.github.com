---
layout: post
title: redis detect_buffers参数
category: 技术
tags: [redis] 
keywords:  redis，参数
---

公司一小伙碰到个问题，要求通过redis传送压缩文件给另外一台机器，压缩文件是二进制的，那么就要求从redis中读写二进制的内容，而这时候就碰到个问题，redis读出的数据默认都是转成string类型的，于是我们就找到了这样一个参数detect_buffers，在建立redis连接时设置，默认为false，打开设置为true，打开这个参数redis技能返回二进制数据了，官方文档如下：

`detect_buffers`: default to `false`. If set to `true`, then replies will be sent to callbacks as node Buffer objects
if any of the input arguments to the original command were Buffer objects.
This option lets you switch between Buffers and Strings on a per-command basis, whereas `return_buffers` applies to
every command on a client.

然而这个地方比较奇葩的是设置redis返回二进制数据的方式，我们的代码是nodejs编写，如果传入的key是以buffer的方式传入，那么得到返回的value就是二进制的，如果传入的参数是string方式传入的，那么返回的value就是string类型，传入参数的类型定义了返回参数的类型。。这个给人的感觉就是redis当初写的时候没有考虑到需要返回二进制类型的数据，加了这个个需求，然后又不想加参数，所以写成了这样。








