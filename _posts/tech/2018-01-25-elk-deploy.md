---
layout: post
title: ELK运维中遇到的问题及解决方案
category: 技术
tags: [Linux, ELK] 
keywords: ELK
---

### 起因

ELK现网部署遇到过很多问题，这里汇总下趟过的坑

### 问题

##### ELK支持集群部署，支持多节点多分片多副本等方式，如何评估怎么配置能够满足当前生产环境的需求？

Elastic Stack方案栈提供rally压测工具，对ELK集群进行压测并给出详尽的压测报告，可使用rally工具自带的数据集也可使用自定义的数据集，结合现网数据量进行评估

##### logstash grok工具正则表达式如何配置调试？

logstash grok正则表达式的编写是一个苦力活，这里推荐一个在线调试工具[grok 在线调试工具](http://grokdebug.herokuapp.com/)可以提高正则调试效率，另外，为了确保日志都能导入到ElasticSearch中，建议采用下面配置

    output {
        if "_grokparsefailure" in [tags] {
            file {
                path => "./test-%{+YYYY-MM-dd}.txt"
                codec => line { format => "%{message}"}
            }
        }else{
            elasticsearch {
                hosts => ["10.12.239.142:9200"]
                user => "logstash_internal"
                password => "endertest"
            }
        }
    }

没有被grok正则解析的日志集中导入到一个文件，定期查看该文件，调整正则表达式

##### 一天日志有多行，在ElasticSearch中如何合并成一行显示?

使用logstash multiline组件

##### 如何监控ELK集群自身的健康状态？

引入x-pack组件，全面监控ELK各个节点的运行状态，并提高ELK集群的安全性

##### 如何向ElasticSearch中导入不同结构的日志？

logstash会创建一个默认的模板导入数据，如果需要导入不同结构的数据，对每个结构的数据建立按日期的索引就需要logstash配置中指定索引使用的模板，具体配置可参考[这篇博客](http://qindongliang.iteye.com/blog/2290384)

##### ElasticSearch JVM内存如何配置?

笔者现网就遇到过导入日志一直延迟，ELK集群运行一天后几乎成瘫痪状态，这里就是ElasticSearch JVM内存设置的问题，默认JVM可用内存只有1G，这对于ElasticSearch来说非常小，运行一段时间以后就一直处于GC状态了，一般建议这个值尽量大，但是不要超过32G，修改jvm.options文件可设置这个值

