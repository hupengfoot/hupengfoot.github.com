---
layout: post
title: 《redis设计与实现》
category: 读书
tags: [Linux] 
keywords: redis
---

手头有个项目需求需要大量用到redis，这个项目做到现在也算有半年的redis重度使用经验了，但是我们也就一直停留在redis基本数据结构使用的阶段，对于redis有哪些高端功能以及redis的运维都不熟，这样不仅用的过程中有一种隔靴搔痒的感觉，而且项目放到现网一旦redis出现问题可能会无从下手解决，因此决定补习一下功课，搞了本《redis设计与实现》来读。

其实之前刚毕业的时候接到的第一个任务就是研究redis，并做了个分享，但当时没有任何使用经验，纯粹看了些网上的资料，并没有找到痛点，这次也算是复习下功课了，这也使我这本书看得相当快，3天就看完了。基本上这本书讲得非常容易懂，这可能也与redis设计本来就很简单有关，大道至简么，看完这本书redis使用，功能，性能，运维方面的知识基本上就都ok了，想要做一些深入hack的同学才需要去读redis的源码，之前读过《深入理解nginx》，该书基本是讲源码的，读过很多nginx源码的我去看的时候依然觉得很难读。

讲讲我对redis的理解吧，redis给我的感觉是一个纯工程实践中生长出来的一个产品，完全针对mysql的磁盘存储和关系型数据库固有的性能缺陷做的架构优化中的一个中间产品，redis提供了丰富的数据结构和操作方式提供给开发者高效的API去处理关系型数据库中非常难处理的操作，redis内存存储的方式极大的提高了后台访问的速度，但同时也带了了数据持久化以及数据存储量的问题，redis对这些问题都给出了不错的解决方案。redis的整个设计理念中主要围绕两个词来进行，效率和折中，因为内存的宝贵性，redis十分强调存储效率，试图以最少的代价存储最多的信息，但是过度追求存储的效率又会影响访问的效率，因此redis提供了一些折中的方案，压缩表就是一个例子。redis的运维和性能调优是很需要经验的，需要根据每个业务场景和发展状况进行相应的调整，redis提供了丰富的可配置参数帮你完成这个事情，需要知道，在不同业务场景下，参数的配置可能对redis的性能产生很大影响。

nosql+mysql应该是目前网站应用的主流架构，redis作为nosql的优秀作品正在被许多公司重度使用，是很值得深入研究的一个东西，看完《redis设计与实现》还只是一个入门。

