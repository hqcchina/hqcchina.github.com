--- 
layout: post
title: "Docker日志自动化: ElasticSearch、Logstash、Kibana以及Logspout"
wordpress_id: 9
wordpress_url: http://www.yunjing.me/?p=9
date: 2015-12-31 11:40:00 +08:00
category: docker
tags: 
 - linux
 - docker
 - elasticsearch
 - logstash
 - kibana
 - logspout
---

## 摘要
日志的分析和监控非常重要，对于日志来说，最常见的需求就是收集、查询、显示，这三个需求 分别对应logstash、elasticsearch、kibana的功能。本文介绍了如何使用ELK Stack来构建一个实时日志查询、收集与分析系统，而在Docker之上构建这个Stack，可以事办功倍。

```
    $ docker run -d -v /data --name dataelk busybox
    $ docker run -p 10040:80 \
       -v /path/to/your/logstash/conf.d:/etc/logstash \
       --volumes-from dataelk \
       wildurand/elk
```

