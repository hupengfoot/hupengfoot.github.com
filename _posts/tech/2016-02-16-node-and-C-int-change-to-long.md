---
layout: post
title: 一个整数拼接引发的血案
category: 技术
tags: [Node] 
keywords:  Node，整数拼接
---

### 起因
我们的系统中有这样一个需求，需要将两个int型拼接起来作为一个ID插入数据库表中，目的是为了能够直接通过ID算出数据库分表号，ID构成为自增ID + 分表ID。然后在我们的系统使用中却连续不断的出现问题~~

### 问题发现及解决
我们的后台采用node实现，首先碰到的问题就和node有关，node只能精确表达小于2的53次方大小的整数，而我们把自增ID放在了拼接ID的高32位，这样实际上自增ID达到2的21次方时就会出现问题（也就2097152），所以我们系统运行一段时间以后就出现了问题，因为ID拼接不正确，导致查找分表不正确。这个问题我们的解决方法是实现了一个node的C模块专门负责ID的拼接，这样两个32位的int，拼接成一个64位的unsigned long，高位自增ID就能支持到21亿了，感觉应该是够了~~

然而系统运行没多久又出现了问题，我们的分表ID可能是一个负数，而我的实现代码如下，
    ```python
#GetUniqueID
unsigned long resultNum = (num1 << 32L) + num2;
    ```
    ```
#GetTopID
unsigned long resultNum = num / 4294967296;
    ```
#GetEndID
int resultNum = num % 4294967296;
    ```
这样如果第一个参数传100，第二个参数传-2，计算出来的ID就是429496729598，反算出来的自增ID是99，分表ID是-2，得到了错误的结果！原因很明显，做整形拼接的时候不应该采用这种方式，计算机采用补码表示整数，这样如果参数有负数就无法还原，解决方法是改成如下写法，
    ```
#GetUniqueID
unsigned long resultNum = (num1 << 32L)|(0xFFFFFFFF & num2);
    ```
    ```
#GetTopID
unsigned long resultNum = (num & 0xFFFFFFFF00000000)>>32;
    ```
#GetEndID
int resultNum = num & 0x00000000FFFFFFFF;
    ```
这样的写法其实还是有问题，第一如果高位ID时负数，依然无法还原，因为我们的系统中高位不会为负数，所以这个问题可以忽略；第二如果地位ID超过2的32次方，也无法还原，这种情况是有可能发生的
