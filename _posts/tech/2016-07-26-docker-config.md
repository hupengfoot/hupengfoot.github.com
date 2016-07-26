---
layout: post
title: mac docker安装配置 
category: 技术
tags: [docker, mac] 
keywords:  docker, mac
---

新做了一个项目想使用docker做发布版本，一是赶时髦，而是确实看到了docker的先进性，于是先自己搭环境玩了起来。

本人的环境是mac x，目前的主流做法是安装 Docker Toolbox，使用docker-machine工具来做docker虚拟机管理。原理如下：由于docker的底层技术用到了linux的特新LXC，所以在mac上运行需要先启动一个linux虚拟机作为宿主机，使用docker-machine工具来管理；然后就可以正常使用各种docker指令了，这个地方有一个坑，有时候使用docker-machine启动虚拟机之后运行docker指令会报各种无法连接到虚拟机的错误，是因为相关环境变量没有设置，运行指令eval "$(docker-machine env default)" 即可设置相应的环境变量
