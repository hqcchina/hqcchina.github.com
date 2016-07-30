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


按照官方Docker镜像的说明启动Docker容器（示例：Docker-Compose）
```
version: '2'
services:
  elasticsearch:
    image: elasticsearch:2.3.4
    ports:
      - "10041:9200"
      - "10042:9300"
    volumes:
      - /data/logging/config:/usr/share/elasticsearch/config
      - /data/logging/data:/usr/share/elasticsearch/data
```


启动elasticsearch的Docker容器会报以下错误：
```
[root@localhost logio]# docker logs 0bcaefb7b341
log4j:WARN No appenders could be found for logger (bootstrap).
log4j:WARN Please initialize the log4j system properly.
log4j:WARN See http://logging.apache.org/log4j/1.2/faq.html#noconfig for more info.
Exception in thread "main" java.lang.IllegalStateException: Unable to access 'path.scripts' (/usr/share/elasticsearch/config/scripts)
Likely root cause: java.nio.file.AccessDeniedException: /usr/share/elasticsearch/config/scripts
        at sun.nio.fs.UnixException.translateToIOException(UnixException.java:84)
        at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:102)
        at sun.nio.fs.UnixException.rethrowAsIOException(UnixException.java:107)
        at sun.nio.fs.UnixFileSystemProvider.createDirectory(UnixFileSystemProvider.java:384)
        at java.nio.file.Files.createDirectory(Files.java:674)
        at java.nio.file.Files.createAndCheckIsDirectory(Files.java:781)
        at java.nio.file.Files.createDirectories(Files.java:767)
        at org.elasticsearch.bootstrap.Security.ensureDirectoryExists(Security.java:337)
        at org.elasticsearch.bootstrap.Security.addPath(Security.java:314)
        at org.elasticsearch.bootstrap.Security.addFilePermissions(Security.java:248)
        at org.elasticsearch.bootstrap.Security.createPermissions(Security.java:212)
        at org.elasticsearch.bootstrap.Security.configure(Security.java:118)
        at org.elasticsearch.bootstrap.Bootstrap.setupSecurity(Bootstrap.java:196)
        at org.elasticsearch.bootstrap.Bootstrap.setup(Bootstrap.java:167)
        at org.elasticsearch.bootstrap.Bootstrap.init(Bootstrap.java:270)
        at org.elasticsearch.bootstrap.Elasticsearch.main(Elasticsearch.java:35)
Refer to the log for complete error details.
```
解决办法：
问题原因在于elasticsearch需要在config目录下生成elasticsearch.yml配置文件，但是读取失败导致报错。

移除对config目录的映射，直接把elasticsearch.yml映射进来就好。



** Step 3: Configuring td-agent **

编辑*/etc/td-agent/td-agent.conf*文件：
```
<match docker.**>
  @type stdout
</match>
```
然后，重启fluentd服务：
```
/etc/init.d/td-agent restart
```


## 参考资料
1、(http://elk-docker.readthedocs.org/#forwarding-logs-filebeat)sebp/elk
2、(https://www.digitalocean.com/community/tutorials/how-to-use-logstash-and-kibana-to-centralize-and-visualize-logs-on-ubuntu-14-04) How To Use Logstash and Kibana To Centralize Logs On Ubuntu 14.04
