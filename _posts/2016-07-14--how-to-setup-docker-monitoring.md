--- 
layout: post
title: "Docker监控方案 - cAdvisor + InfluxDB + Grafana"
wordpress_id: 19
wordpress_url: http://www.yunjing.me/?p=19
date: 2016-07-14 10:09:00 +08:00
category: docker
tags: 
 - docker
 - monitoring
 - cadvisor
 - influxdb
 - grafana
---

![夸张的监控示意图](/files/2016/07/war_games_movie.jpg)

## Docker监控组件介绍

[cAdvisor](https://github.com/google/cadvisor) - 一个Google出品的开源容器监控工具。只需在宿主机上部署cAdvisor容器，用户就可通过Web界面或REST服务访问当前节点和容器的性能数据（CPU、内存、网络、磁盘及磁盘占用等）。默认cAdvisor是将数据缓存在内存中，数据展示能力有限；它也提供不同的持久化存储后端支持，可以将监控数据保存、汇总到Google BigQuery、InfluxDB或Redis上。用户可以进一步加工处理这些监控指标，实现数据展现、报警、基于规则的自动化执行的能力。

[InfluxDB](https://github.com/influxdata/influxdb) - 一个强大好用的时间序列(time series)数据库，可以非常简单地利用类SQL的方式处理时序数据。

[Grafana Metrics Dashboard](https://github.com/grafana/grafana) - 一个流行的监控仪表盘应用，可以非常友好的展现数据信息，页面相当酷炫。


## 安装过程

1. 安装InfluxDB 0.13版本

```
$ docker run -p 8083:8083 -p 8086:8086 --name influxdb -d influxdb:0.13-alpine
```

打开 http://DockerIP:8083 检查一下InfluxDB是否安装成功
![InfluxDB管理后台界面](/files/2016/07/influxdb_dashboard2.jpg)

2. 创建cAdvisor数据库



[原文链接](https://www.brianchristner.io/how-to-setup-docker-monitoring/)