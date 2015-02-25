---
layout: post
title: node express大文件上传问题
category: 技术
tags: [node,express,大文件上传] 
keywords: node,express,大文件上传 
---

### 问题
用node写了个文件上传服务器，实现读取前端提交的上传表单，将上传内容上传到公司公有云文件服务器上，服务器的架构是通过nginx做反向代理，将/uploadfile路径的请求转给node写的文件上传服务器，node进程负责处理具体的上传逻辑，node进程采用express框架处理请求，环境搭建好后测试发现，小文件上传是ok的，大文件上传会返回超时。

### 解决方法
问题判断肯定是超时时间的问题，先把nginx upstream超时时间加大，设置proxy_read_timeout为3600s，测试发现任然不解决问题，大文件上传依然返回失败，nginx upstream 超时都设这么大了，还是超时，太不科学了，于是看node的日志，发现node每过一段时间就会打印一个60000ms超时的日志，原来node express自己设置了一个res的超时时间，因此需要修改express的超时时间，修改后测试，上传大文件成功，问题解决。

### 总结
这个问题在实际解决中比上面写的要费神很多，主要是实际环境中nginx的配置比较复杂，一直把问题定位为nginx upstream超时配置的问题，各种尝试更改nginx的配置，搞了很久才发现是node express的超时。

