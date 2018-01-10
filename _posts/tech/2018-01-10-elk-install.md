---
layout: post
title: 现网环境elk部署方案
category: 技术
tags: [Linux, elk]
keywords: 日志,elk
---

### 起因
现网环境排查bug经常需要查询现网日志，我们现有的现网日志收集系统虽然实现了现网各台业务机器日志的集中管理，但是依然需要登录到现网日志收集机器，通过linux系统指令全文检索日志进行故障排查，这里就存在两个问题，一是公司IDC环境和开发环境有网络隔离，需要登录跳板机再登陆日志机器进行查询，二是全文检索日志效率低下，影响排查问题的效率。

ELK是一套成熟的日志收集检索展示方案，能够然我们直接在页面上对日志进行检索，并对日志进行了结构化索引，提高日志检索效率，能够很好的解决上述我们的两个痛点，并且提高强大的自定义图标功能及告警功能，能够提供提高对现网运行状况的把控能力更多的想象空间。

### ELK的部署
首先介绍一下这里涉及的组件

[elasticsearch 版本6.1.1](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.1.zip) ELK的核心系统，一个高可扩展的全文索引和检索引擎，数据的存储和管理都在这个组件中实现。

[kibana 版本6.1.1](https://artifacts.elastic.co/downloads/kibana/kibana-6.1.1-linux-x86_64.tar.gz) 检索结果展示系统，实际上就是调用elasticsearch提供的接口，并对用户提供友好的图标界面的web系统。

[logstash 版本6.1.1](https://artifacts.elastic.co/downloads/logstash/logstash-6.1.1.zip) 日志收集系统，负责搜集和结构化数据，并将结构化的数据传入elasticsearch。

[x-pack 版本6.1.1](https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-6.1.1.zip) x-pack是ELK工具栈中的一个强大工具，提供了很多功能，我们这里主要是用到x-pack的监控功能，对elasticssearch、kibana、logstash三个组件运行的健康状况进行监控上报，并在kibana中展示。

下面先介绍一下我们ELK的部署结构

![alt](http://shp.qpic.cn/zc_large/0/558_1515575154000/0)

我们现网业务日志通过UDP传输给一个日志服务器，实现日志的统一收集，然后在此基础上，在日志服务器部署logstash读取收集到的日志文件，转发给ELASTIC+KIBANA服务器，将日志结构化写入ELASTIC的索引，然后通过KIBANA的界面展示给用户

#### ELASTICSEARCH部署

首先安装jdk环境，必须安装jdk-1.8以上的版本，我们这里使用的 [jdk-8u151-linux-x64](http://download.oracle.com/otn-pub/java/jdk/8u151-b12/e758a0de34e24606bca991d704f6dcbf/jdk-8u151-linux-x64.tar.gz)

然后下载 [elasticsearch-6.1.1](https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-6.1.1.zip) , 这里需要注意，因为elasticsearch可以执行任何传入的命令，所以要求必须已非root用户启动，先创建一个新用户，如:elk，然后修改配置文件如下

    #集群名称
    cluster.name: xianyou_log
    #节点名称
    node.name: node-1
    #数据存储目录
    path.data: /data/elasticData
    #elasticsearch 日志存储目录
    path.logs: /data/logs
    #必须这样配置，否则无法启动
    bootstrap.memory_lock: false
    bootstrap.system_call_filter: false
    #监听IP
    network.host: 0.0.0.0
    #监听端口
    http.port: 9200
    #集群存在的主机
    discovery.zen.ping.unicast.hosts: ["xx.xx.xx.xx"]

然后下载 [x-pack-6.1.1](https://artifacts.elastic.co/downloads/packs/x-pack/x-pack-6.1.1.zip), 到elasticsearch的代码根目录下执行`./bin/elasticsearch-plugin install file:///PATH/x-pack-6.1.1.zip`，使elasticsearch集成x-pack的功能，然后就可以切换到elk用户下，执行`./bin/elasticsearch`启动elasticseach了，这里还有可能报一个系统参数设置的问题，elasticsearch启动对内存使用有一定要求，这里按照提示修改系统参数设置再启动就OK了

elasticsearch启动后还需在代码根目录下执行`./bin/x-pack/setup-passwords auto`生成一组elasticsearch的用户用于x-pack的监控上报功能，这些用户名密码在配置kibana和logstash时会用到，最后记录在一个文件中方便查阅

    Changed password for user kibana
    PASSWORD kibana = QrawdIH&%JY6z&=@rNfr

    Changed password for user logstash_system
    PASSWORD logstash_system = ender@idc

    Changed password for user elastic
    PASSWORD elastic = kD6@GnQJPlcAPrT%-ZWP

#### KIBANA部署

下载 [kibana-6.1.1](https://artifacts.elastic.co/downloads/kibana/kibana-6.1.1-linux-x86_64.tar.gz)，在kibana代码更目录下执行`./bin/kibana-plugin install file:///PATH/x-pack-6.1.1.zip`安装x-pack，集成x-pack的功能，然后修改kibana的配置文件

    #kibana监听port
    server.port: 5601
    #kibana监听IP
    server.host: "xx.xx.xx.xx"
    #访问的elasticsearch的地址
    elasticsearch.url: "http://localhost:9200"
    #访问elasticsearch的用户名
    elasticsearch.username: "kibana"
    #访问elasticsearch的密码
    elasticsearch.password: "QrawdIH&%JY6z&=@rNfr"

然后使用`./bin/kibana`命令启动kibana，这是用就可以通过访问`http://xx.xx.xx.xx:5601/`访问kibana，并查看elasticsearch集群状态了，但是这时候elasticsearch集群中并没有有效的数据，如果需要的话可以再配置一个nginx，实现80端口访问。

#### logstash的部署
在日志收集服务器上下载 [logstash-6.1.1](https://artifacts.elastic.co/downloads/logstash/logstash-6.1.1.zip)，在代码根目录下执行`./bin/logstash-plugin install file:///PATH/x-pack-6.1.1.zip`,集成x-pack的功能，然后按照需要解析的日志格式编写一个logstash的日志解析配置文件first-pipeline.conf

    input {
        #命令行输入
        #stdin{}
        #文件输入
        file {
            #使用codec组件合并多行日志
            codec => multiline {
                pattern => "^\["
                negate => true
                what => "previous"
            }
            #指定文件路径
            path => "/data/newGamer/zc_svr/logs/www*.out"
            #定义索引类型
            type => "backend-nginx"
        }
    }
    filter {
        #使用grok组件结构化日志
        grok {
            #match => { "message" => "%{COMBINEDAPACHELOG}"}
            match => { "message" => "\[%{TIMESTAMP_ISO8601:timestamp}\](\s)+%{LOGLEVEL:log_level}(\s)+-(\s)+\[%{NOTSPACE:code_line}\](\s)+(\[SN:(?<SN>\d+)\])?\s+(\[iQQ:(?<iQQ>\d+)\]\s+)?"}
        }
        #解析客户端IP
        geoip {
            source => "clientip"
        }
        #没有该配置，一条日志的写入事件会被替换成写入elasticsearch的时间
        date {
            match => ["timestamp", "yyyy-MM-dd HH:mm:ss.SSS"]
            target => "@timestamp"
        }
    }
    output {
        #结果输出到终端
        #stdout { codec => rubydebug }
        #结果输出到elasticsearch
        elasticsearch {
            hosts => ["10.12.239.142:9200"]
            user => "logstash_internal"
            password => "endertest"
        }
    }   

这里需要注意，要登录带kibana管理界面去手动创建logstash_internal用户，用于写入日志，如下

![alt](http://shp.qpic.cn/zc_large/0/350_1515578628000/0)

然后还需要修改logstash.yml文件，配置如下

    xpack.monitoring.elasticsearch.url: ["http://xx.xx.xx.xx:9200"]
    xpack.monitoring.elasticsearch.username: logstash_system
    xpack.monitoring.elasticsearch.password: ^@SPS$NO@OZCSpOX8ERs

用于实现x-pack的监控上报功能，这里需要注意的是logstash_system用户的这个自动生成的密码用时候不对（不知道为啥），可以在kibana界面手动再设置一次，然后使用命令`./bin/logstash -f config/first-pipeline.conf`启动logstash，就可以实现日志收集和logstash运行状态上报的工作，效果图如下

![alt](http://shp.qpic.cn/zc_large/0/844_1515579028000/0)
![alt](http://shp.qpic.cn/zc_large/0/346_1515579073000/0)

### 总结
ELK是日志收集的一个较为成熟的系统，提供了集群高可扩展方案，自我监控能力和并自带压测工具，ELK解决了现网查日志不方便的痛点，并且可以自定义图表，更加直观的展现现网运行的健康状况，提供了现网监控的更多的可能性

### 参考文档
[ELK官网](https://www.elastic.co/guide/index.html)
[grok内置正则](https://github.com/elastic/logstash/blob/v1.4.2/patterns/grok-patterns)
