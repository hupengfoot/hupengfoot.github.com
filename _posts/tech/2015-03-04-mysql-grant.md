---
layout: post
title: mysql grant的问题
category: 技术
tags: [Linux] 
keywords:  mysql, grant
---

给mysql数据库添加用户访问权限使用`Grant 操作 on 数据库名.* to 用户名@'IP'  identified by '用户密码';`命令，例如：`Grant select on db10020.* to AUSER@'10.140.149.29'  identified by 'APASSWORD';`，表示允许用户AUSER从IP 10.140.149.29访问数据库的10020，只允许select操作，用户密码为APASSWORD。

但是有时候运行该命令会报如下错误`ERROR 1044 (42000): Access denied for user 'root'@'%' to database 'db10020'`，通过`show grants`命令，我们可以看到`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9'`，说明root用户没有grant权限，不能分配给其它用户权限，需要修改mysql.user表的对应的user的Grant_priv项为Y，然后flush PRIVILEGES，登出后再登入改root用户就可以给其它用户分配权限了，此时`show grants`结果为`GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY PASSWORD '*6BB4837EB74329105EE4568DDA7DC67ED2CA2AD9' WITH GRANT OPTION`。

修改数据库访问权限这种不常用操作特别容易忘，每次查都要花很多时间，这里自己mark一下。
