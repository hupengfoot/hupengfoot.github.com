---
layout: post
title: graphql查询方案优化
category: 技术
tags: [graphql, schema]
keywords: graphql, dataloader
---

### 起因
graphql API对外提供完整的接口文档，按需取数据的功能，对于接口联调和前端开发同学看来绝对是神器，但对于初看graphql的后台同学看来，这个取数据怎么取？对于一个两层结构的列表数据，我对列表中的每一项都取一次二级数据？这后台抗得住吗？

举个例子：我们有这样一个应用场景，我们要取一个用户列表，列表中每一项展示有3个字段id、name、friends，friends字段是一个数组，结果集中要做二层展开，展开成id、name，结果集如下:

    {
        "data": {
            "userList": [
                {
                    "id": "1",
                    "name": "Dan",
                    "friends": [
                        {
                            "id": "2",
                            "name": "Marie"
                        },
                        {
                            "id": "3",
                            "name": "Jessie"
                        }
                    ]
                },
                {
                    "id": "2",
                    "name": "Marie",
                    "friends": [
                        {
                            "id": "1",
                            "name": "Dan"
                        }
                    ]
                },
                {
                    "id": "3",
                    "name": "Jessie",
                    "friends": [
                        {
                            "id": "1",
                            "name": "Dan"
                        }
                    ]
                }
            ]
        }
    }

DB中数据结构如下，我们这里使用json文件存储数据，

用户信息表

    {
        "1": {
            "id": "1",
            "name": "Dan"
        },
        "2": {
            "id": "2",
            "name": "Marie"
        },
        "3": {
            "id": "3",
            "name": "Jessie"
        }
    }

用户关系表

    {
        "1":[
            2,
            3
        ],
        "2":[
            1
        ],
        "3":[
            1
        ]
    }

DB层对外提供fakeDB.getAll获取用户列表接口，fakeDB.getFriendsInfo获取用户朋友信息接口，HTTP层再做一层封装，对外提供userBiz.getAll获取用户列表接口，userBiz.getFriendsInfo获取用户朋友信息接口，这样graphql中阶层获取数据请求数据的方式是先调用userBiz.getAll接口获取用户列表，再依次对每个用户调用一遍userBiz.getFriendsInfo接口获取列表每一项的friends字段，日志输出结果如下:

    get all
    get friends info1
    get friends info2
    get friends info3

这样列表项有多少项，就需要发起多少个userBiz.getFriendsInfo接口的HTTP调用，如果返回数据列表项再加大，数据层级再加深，HTTP调用的数量将会变得非常大，有没有办法减少HTTP调用呢，graphql实际上已经提供了解决方案

### 解决方案
graphql引入dataloader组件，提供合并请求的能力，dataloader的原理很简单，他利用nodejs的next tick机制，需要额外后台额外增加一个批量获取用户朋友信息的HTTP接口，修改graphql的schema，使其使用dataloader的方式，dataloader的具体配置详见附件代码，日志输出结果如下:

    get all
    get batch friends info 1,2,3

可见对列表项中的每一项获取friends信息合并成了一个HTTP请求

### 总结
这里假设的场景是一个graphql中间层调用HTTP接口的合并，如果使用graphql作为server端接口标准，同样可以使用dataloader合并sql层的访问

