---
layout: post
category : summary
tagline: "技术栈总结"
tags : [ summary ]
---

{% include JB/setup %}

##RFC

[RFC6020-YANG]
[RESTConf]
[NETConf]


[RFC2119-关键词解释](http://www.ietf.org/rfc/rfc2119.txt)
[RFC2616-HTTP-1.1]()
RFC 5405
[RFC3986]
[RFC4844]
[RFC3629]UTF-8
[RFC4844]
[RFC4741]

##语言

* c cpp
* python ruby
* lua
* java
* javascript
* html css
* R
* haskell
* lisp

##数据库与存储

* mysql
* mongodb
* couchDB
* cassandra
* mdb
* Leveldb
* swift
* ceph
* redis
* memcache

##系统

* 进程
* 线程
* 内存管理
* 协程
* socket
* IO
* 网络
* 存储
* 其他,看 APUE

##服务器

* apache
* nginx
* nodejs

##协议

* HTTP(1.0 1.1 2.0)
* YANG
* NETCONF
* RTSP
* NETCONF

##RPC

* gRPC
* Thrift
* Wildfly
* Dubbo
* JBoss EAP

##MQ

##配置格式

* yaml
* xml
* json
* ini

##日志

* logback
* log4j
* zlog

##错误处理

* try catch
* assert

##信号处理

* 支持哪些信号，屏蔽哪些信号

##文档

* droxgen
* swagger [文档 UI](https://github.com/swagger-api)

##工具

* screen
* ssh
* tmux
* git
* markdown : mdp landslide
* 其他运维相关工具

##云计算

* Openstack
* Mesos + Marathon
* Docker

##Docker 管理工具

* Swarm
* Kebernetes
* weave
* socketplane

##框架

* Zookepper



##视频

* openCV
* uv4l

##API 设计
http://www.ruanyifeng.com/blog/2014/05/restful_api.html
http://www.thoughtworks.com/insights/blog/rest-api-design-resource-modeling
http://codeplanet.io/principles-good-restful-api-design/
https://restful-api-design.readthedocs.org/en/latest/

##实时日志系统
基于流式处理，采用 flume 收集日志，发送到 kafka 队列做缓冲，storm 分布式实时框架进行消费处理，
短期数据落地到 hbase、mongo中，长期数据进入 hadoop 中存储
