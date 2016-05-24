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
$ docker run -p 5601:5601 -p 9200:9200 -p 5044:5044 -p 5000:5000 -it --name elk sebp/elk
```

## 问题解答
Q: Failed to tls handshake with x.x.x.x x509: cannot validate certificate for x.x.x.x because it doesn't contain any IP SANs
A: 按以下步骤重新生成CA证书：
1）打开/etc/ssl/openssl.cnf文件，查找v3_ca字段，接以下格式新增subjectAltName字段；
```
[v3_ca]
subjectAltName = IP: logstash_server_private_ip
```
2）重新生成CA证书；
```
cd /etc/pki/tls
sudo openssl req -config /etc/ssl/openssl.cnf -x509 -days 3650 -batch -nodes -newkey rsa:2048 -keyout private/logstash-beats.key -out certs/logstash-beats.crt
```

## 参考资料
1、(http://elk-docker.readthedocs.org/#forwarding-logs-filebeat)sebp/elk
2、(https://www.digitalocean.com/community/tutorials/how-to-use-logstash-and-kibana-to-centralize-and-visualize-logs-on-ubuntu-14-04) How To Use Logstash and Kibana To Centralize Logs On Ubuntu 14.04
