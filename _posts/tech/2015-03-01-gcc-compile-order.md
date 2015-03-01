---
layout: post
title: gcc链接库顺序
category: 技术
tags: [Linux] 
keywords:  gcc
---

公司一小伙碰到个编译链接的问题搞不定找我帮忙，作为一个乐于助人的前浪，我挺喜欢帮人解决问题的，问题大概是这样的，该小伙要把一个公司的C代码写的组件整合到node中使用，需要编译一个C模块，按照教程整合编译好以后运行demo，报错undefined symbol xxxxx，但是直接运行C的demo是ok的，这种报错是常见的链接问题，这样问题就基本定位了。

node的C模块实际上就是编译一个.node后缀的动态链接库供node使用，于是我先用readelf查看他编译出来的.node文件的符号表，发现他报undefined symbol的符号虚拟地址项填的0，这显然是链接得时候没有找到符号定义，于是我不用node自己的链接方式，试着自己写gcc做链接，去链接他要整合的模块的库，他要整合的模块的库有两个，姑且先叫aaa，bbb，编译命令gcc -o test.node -laaa -lbbb -share -IPIC，用readelf查看编译出的test.node，发现报undefined的符号虚拟地址项还是填的0，然后想起gcc做链接时符号是从左向右从文件中读入的，符号定义也是从左香油找的，因此链接过程和写命令顺序有关，试着调整了一下gcc命令，gcc -o test.node -lbbb -laaa -share -IPIC，编译好后用readelf查看，果然ok了，报undefined的符号虚拟地址项有了值，运行demo通过，看来是引文bbb库有对aaa库的依赖所导致的问题，又帮人搞定了个问题，挺爽的。

不过我又想，这尼玛要是有N多个依赖库，我又不知道它的依赖关系不是要搞死了，于是查了查资料，发现gcc还是考虑到了这个问题，gcc有个Xlinker参数，可以使用Xlinker参数指示gcc循环查找依赖，这样会减慢编译链接的速度，但是能解决链接库依赖关系的问题，因此一般已知依赖关系的话就不要使用这个参数了。另外，gcc写链接库的一个原则就是越是基础的库就越往命令的后面写。 


