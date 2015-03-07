---
layout: post
title: mysql中文乱码问题
category: 技术
tags: [Linux] 
keywords: mysql, 中文, utf8 
---

这几天看线上服务器的mysql数据满版满版的问号终于受不了了，决定解决一下这个问题。

线上mysql的状况是这样的，我们的node服务器可以正常使用mysql读写中文，但是使用mysql客户端去连接mysql查看数据时，所有的中文均显示问号，虽然不影响日常使用，但总会多多少少带来一些不方便，而且我尝试用python从改mysql中读取数据，发现中文也会乱码，这个问题就有点严重了，因为我们会使用pyhton写一些离线处理脚本。

这个问题显然是字符串编码的问题，使用mysql命令`SHOW VARIABLES LIKE 'character%'`查看mysql指定的编码格式如下:

    character_set_client latin1
    character_set_connection latin1
    character_set_database utf8
    character_set_filesystem binary
    character_set_results latin1
    character_set_server utf8
    character_set_system utf8

可以看到character_set_results指定的编码格式为latin1，character_set_results含义是mysql传输给client的编码格式，我们的中文数据以utf8格式存入数据库，以latin1的格式传给client，导致client不能争正确解析中文，因此显示问号，解决方法是将character_set_results设置为utf8，使用命令`set NAMES utf8`，设置完后mysql client中文显示正确。

但是这里问题还没有完全解决，该命令只能保证这一个client正确显示中文，对其他的连接不生效，我们需要修改配置文件my.cnf，在[mysqld]添加如下两行：

    init-connect='SET NAMES utf8'
    character-set-server=utf8

然后重启mysql，问题得到解决。后来换来台服务器，使用同样的方法设置发现中文显示仍然不正确，后来发现是mysql版本的问题，mysql5.5的版本使用如下命令设置：

    init_connect='SET collation_connection = utf8_unicode_ci' 
    init_connect='SET NAMES utf8' 
    character-set-server=utf8 
    collation-server=utf8_unicode_ci 
    skip-character-set-client-handshake

5.6版本的mysql则如上面得方法设置，其他版本的没有做过测试。

不过这里依然还有个遗留问题，为啥character_set_results Latin1时node能正确读取中文，而python不能？
