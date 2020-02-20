---
title: EFK部署(单机)
date: 2019-03-12 11:12:56
tags: 
  - 环境搭建
  - EFK
toc: true 
categories: 
  - 环境搭建
comments: true
---

## 1.介绍

EFK不是一个软件，而是一套解决方案，并且都是开源软件，之间互相配合使用，完美衔接，高效的满足了很多场合的应用，是目前主流的一种日志系统。EFK是三个开源软件的缩写，分别表示：Elasticsearch , FileBeat, Kibana , 其中ELasticsearch负责日志保存和搜索，FileBeat负责收集日志，Kibana 负责界面,当然EFK和ELK只有一个区别，那就是EFK把ELK的Logstash替换成了FileBeat，因为Filebeat相对于Logstash来说有2个好处：

1、侵入低，无需修改程序目前任何代码和配置
2、相对于Logstash来说性能高，Logstash对于IO占用很大

当然FileBeat也并不是完全好过Logstash，毕竟Logstash对于日志的格式化这些相对FileBeat好很多，FileBeat只是将日志从日志文件中读取出来，当然如果你日志本身是有一定格式的，FileBeat也可以格式化，但是相对于Logstash来说，还是差一点。

<!--more-->

**Elasticsearch**

Elasticsearch是个开源分布式搜索引擎，提供搜集、分析、存储数据三大功能。它的特点有：分布式，零配置，自动发现，索引自动分片，索引副本机制，restful风格接口，多数据源，自动搜索负载等。

**FileBeat**

Filebeat隶属于Beats。目前Beats包含六种工具：
Packetbeat（搜集网络流量数据）
Metricbeat（搜集系统、进程和文件系统级别的 CPU 和内存使用情况等数据）
Filebeat（搜集文件数据）
Winlogbeat（搜集 Windows 事件日志数据）
Auditbeat（ 轻量型审计日志采集器）
Heartbeat（轻量级服务器健康采集器）

**Kibana**

Kibana可以为 Logstash 、Beats和 ElasticSearch 提供的日志分析友好的 Web 界面，可以帮助汇总、分析和搜索重要数据日志。

## 2.环境准备

CentOS 7.0 64位 4核8G

jdk1.8

EFK下载地址 <https://www.elastic.co/downloads>

我这里用的都是6.5.1最新稳定版本 elasticsearch-6.5.1,filebeat-6.5.1,kibana-6.5.1

## 3.JDK 配置

Elasticsearch需要运行在Java 1.8 及以上，所以需要先安装Java1.8

我这里是下载好的tar包,放在了/usr/local 目录下,解压并配置环境变量

```
tar -xzvf  jdk-8u191-linux-x64.tar.gz

vim /etc/profile

export JAVA_HOME=/usr/java/jdk1.8.0_141
export CLASSPATH=$JAVA_HOME/lib/tools.jar:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib
export PATH=$JAVA_HOME/bin:$PATH
```

按 Esc 键退出编辑模式，输入 :wq 保存并关闭文件。
加载环境变量：

```
source /etc/profile
```

查看 jdk 版本。当出现 jdk 版本信息时，表示 JDK 已经安装成功。

```
java -version
java version "1.8.0_191"
Java(TM) SE Runtime Environment (build 1.8.0_191-b12)
Java HotSpot(TM) 64-Bit Server VM (build 25.191-b12, mixed mode)
```



## 4.安装elasticsearch

我这里新建了一个文件夹去管理EFK

### 目录结构

```
├── efk
│   ├── elasticsearch-6.5.1
│   │   ├── bin
│   │   ├── config
│   │   ├── data
│   │   ├── lib
│   │   ├── LICENSE.txt
│   │   ├── logs
│   │   ├── modules
│   │   ├── NOTICE.txt
│   │   ├── plugins
│   │   └── README.textile
│   ├── filebeat-6.5.1-linux-x86_64
│   │   ├── data
│   │   ├── fields.yml
│   │   ├── filebeat
│   │   ├── filebeat.reference.yml
│   │   ├── filebeat.yml
│   │   ├── kibana
│   │   ├── LICENSE.txt
│   │   ├── logs
│   │   ├── module
│   │   ├── modules.d
│   │   ├── NOTICE.txt
│   │   └── README.md
│   ├── kibana-6.5.1-linux-x86_64
│   │   ├── bin
│   │   ├── config
│   │   ├── data
│   │   ├── LICENSE.txt
│   │   ├── logs
│   │   ├── node
│   │   ├── node_modules
│   │   ├── NOTICE.txt
│   │   ├── optimize
│   │   ├── package.json
│   │   ├── plugins
│   │   ├── README.txt
│   │   ├── src
│   │   └── webpackShims
└── testlog
    ├── a.out
    └── catalina.2018-11-28.log

```

### 创建用户

由于Elasticsearch不能使用root用户打开，所以需要专门创建一个用户来启动Elasticsearch并赋予权限

```
groupadd efk

useradd efk -g efk -p efk

chown -R efk:efk EFK
```

### 创建data数据目录和日志目录

```
su efk

mkdir data/esdata 

mkdir alllogs/eslog
```

### 解压elasticsearch

```
tar -xzvf elasticsearch-6.5.1.tar.gz
```

进入elasticsearch主目录

```
vim config/elasticsearch.yml
```

因为是单机 所以只配了下面几个属性

```
http.port: 9200
path.data: /myproject/efk/elasticsearch-6.5.1/data
path.logs: /myproject/efk/elasticsearch-6.5.1/logs
network.host: 0.0.0.0
```

### elasticsearch控制台中文乱码和jvm大小调节乱码问题

修改conf下面的jvm.options如下

```
-Xms256m
-Xmx256m
# ensure UTF-8 encoding by default (e.g. filenames)
#-Dfile.encoding=UTF-8
-Dfile.encoding=GBK
```

### es主要配置说明

```
#集群主节点名称

cluster.name: mymaster   

node.name: es-1

 # 是否为master

node.master: true

# 是否为数据节点

node.data: true

# 数据目录

path.data: /myproject/efk/elasticsearch-6.5.1/data

# 日志目录

path.logs: /myproject/efk/elasticsearch-6.5.1/logs

# 本机IP

network.host: 0.0.0.0

# 本机http端口

http.port: 9200

# 指定集群中的节点中有几个有master资格的节点

discovery.zen.minimum_master_nodes: 1

# 指定集群中其他节点的IP

discovery.zen.ping.unicast.hosts: ["10.168.1.44","10.168.0.126","10.168.0.127","10.168.0.128","10.168.0.130"]

```



### 启动Elasticsearch

```
./bin/elasticsearch
```

### 后台启动

```
./elasticsearch -d
```

### 查找ES进程

```
ps -ef | grep elastic
```

### 杀掉ES进程

```
kill -9 2382（进程号）
```

### 常见问题

如果遇到错误：max file descriptors [65535] for elasticsearch process is too low, increase to at least [65536]

vi /etc/security/limits.conf  

如果有 * soft nofile 65535 * hard nofile 65535 则将65535修改为65536，如果没有则在后面添加，注意此处的65535对应descriptors [65535]中的65535，修改后的值65536对应increase to at least [65536]，所以当提示不一致时，需要根据具体的错误提示具体修改

如果遇到错误： max virtual memory areas vm.max_map_count [65530] is too low, increase to at least [262144]

```
vi /etc/sysctl.conf
```

添加配置

```
vm.max_map_count=262144
```

并执行命令

```
sysctl -p
```

以上2个修改需要在root用户权限修改，如果是使用xshell开两个窗口的话修改完成之后一定要断开重新登录一下，启动成功用执行命令

```
curl 127.0.0.1:9200  
```

会得到类似以下json

```
{
  "name" : "JEBFbUB",
  "cluster_name" : "elasticsearch",
  "cluster_uuid" : "tDw_Tk9rSyevhkazjbVa0Q",
  "version" : {
    "number" : "6.5.1",
    "build_flavor" : "default",
    "build_type" : "tar",
    "build_hash" : "8c58350",
    "build_date" : "2018-11-16T02:22:42.182257Z",
    "build_snapshot" : false,
    "lucene_version" : "7.5.0",
    "minimum_wire_compatibility_version" : "5.6.0",
    "minimum_index_compatibility_version" : "5.0.0"
  },
  "tagline" : "You Know, for Search"
}
```

## 5.安装kibana

### 解压

```
tar -xzvf kibana-6.5.1-linux-x86_64.tar.gz
```

### 配置

```
vim config/kibana.yml
```

### 添加以下配置或者取消注释并修改

```
logging.dest: /myproject/efk/kibana-6.5.1-linux-x86_64/logs/kibana.log
elasticsearch.url: "http://127.0.0.1:9200"
server.host: "0.0.0.0"
kibana.index: ".kibana"
```

其中elasticsearch.url为Elasticsearch的地址，server.host默认是localhost，如果只是本地访问可以默认localhost，如果需要外网访问，可以设置为0.0.0.0

### 如果外网不能访问,开放端口号

```
firewall-cmd --zone=public --query-port=80/tcp
#查看5601是否占用
firewall-cmd --zone=public --query-port=5601/tcp
#添加5601
firewall-cmd --zone=public --add-port=5601/tcp --permanent
#重载
firewall-cmd --reload
#再次查看5601是否占用
firewall-cmd --zone=public --query-port=5601/tcp
```

### 后台启动

```
nohup ./bin/kibana > logs/kibana.log &
```

### 查找ES进程

```
ps -ef | grep node
```

### 杀掉ES进程

```
kill -9 2382
```

### 外网访问

http://10.146.247.161:8080

## 6. FileBeat安装

### 解压

```
tar -xzvf filebeat-6.5.1-linux-x86_64.tar.gz
```

### 配置

```
vim filebeat.yml
filebeat.inputs:
- type: log
  enabled: true  #默认此处为false  要改成true才会生效 
  paths:
    - /myproject/testlog/*.log
    - /myproject/testlog/*.out
  multiline.pattern: ^\[
  multiline.negate: true
  multiline.match: after
setup.kibana:
  host: "127.0.0.1:5601"
output.elasticsearch:
  hosts: ["127.0.0.1:9200"]
```

配置一定要注意格式，是以2个空格为子级，里面的配置都在配置文件中，列出来的只是要修改的部分，**enabled默认为false，需要改成true才会收集日志。**其中/myproject/test/*.log修改为自己的日志路径，注意-后面有一个空格，如果多个路径则添加一行，一定要注意新行前面的4个空格，multiline开头的几个配置取消注释就行了，是为了兼容多行日志的情况，setup.kibana中的host取消注释，根据实际情况配置地址，output.elasticsearch中的host也一样，根据实际情况配置

### 启动filebeat

```
./filebeat -e -c filebeat.yml 
```

常用的filebeat命令：

-E, --E "SETTING_NAME=VALUE"

覆盖特定的配置设置。 您可以指定多个覆盖。 例如： 

```
 filebeat -E "name=mybeat" -E "output.elasticsearch.hosts=["http://myhost:9200"]"
```

此设置适用于当前正在运行的Filebeat进程。 Filebeat配置文件不会更改。

-c, --c FILE

指定用于Filebeat的配置文件。 你在这里指定的文件是相对于path.config。 如果未指定-c标志，则使用默认配置文件filebeat.yml。

-d, --d SELECTORS

启用对指定选择器的调试。 对于选择器，可以指定逗号分隔的组件列表，也可以使用-d“*”为所有组件启用调试。 例如，-d“publish”显示所有“publish”相关的消息。

-e, --e

记录到stderr并禁用syslog /文件输出。

-v, --v

记录INFO级别的消息。

 

./filebeat -configtest 测试配置文件

./filebeat -httpprof[(host)]:(port):启动http服务器进行性能分析

-memprofile (output file) :将存储器配置文件写入指定的输出文件

-path.config : 设置配置的默认位置

-path.data : 设置数据文件的默认位置

-path.home : 设置其他文件的默认位置

-path.logs : 设置日志文件的默认位置

 

测试filebeat启动后，查看相关输出信息：

./filebeat -e -c filebeat.yml -d "publish"

### 后台方式启动filebeat：

```
nohup ./filebeat -e -c filebeat.yml > logs/filebeat.out &
```

### 查找进程ID并kill掉：

```
 ps -ef |grep filebeat
 kill -9 2233
```

/usr/local/filebeat/filebeat.yml为filebeat 的配置文件地址，需要根据实际情况修改，启动后FileBeat就会自动收集日志了

## 7.配置Kibana

进入 http://10.146.247.161:8080

点击Management进入配置

点击进入Index Patterns

FileBeat默认创建的Elasticsearch索引格式为filebeat-版本号-日期

在第一个红框的输入框中输入

filebeat-6.5.1-*

能匹配到Elasticsearch的索引，第二个红框出会显示出Elasticsearch中已有的索引，点击Next step进入下一步,

点击Create index pattern完成配置，配置完成后点击 Discover就能查看日志了，还能搜索

### 使用

通过Filebeat读取配置目录下的log文件,从kibana页面搜索结果

e.g

```
cd /myproject/test/

vim a.log

teststststststtststs
```

wq保存之后 在kibana查询

teststststststtststs即可查询到结果

EFK搭建完毕.

EFK使用方式后续补充
