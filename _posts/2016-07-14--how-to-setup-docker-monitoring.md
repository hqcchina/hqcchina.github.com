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


## Docker-Compose部署

```
version: '2'
services:
  influxdata:
    image: busybox
    volumes:
      - ./data/influxdb:/data

  influxdb:
    image: tutum/influxdb:0.13
    restart: always
    environment:
      - PRE_CREATE_DB=cadvisor
    ports:
      - "8083:8083"
      - "8086:8086"
    expose:
      - "8090"
      - "8099"
    volumes_from:
      - "influxdata"

  cadvisor:
    image: google/cadvisor:v0.23.2
    links:
      - influxdb:influxdb
    command: -storage_driver=influxdb -storage_driver_db=cadvisor -storage_driver_host=influxdb:8086 -docker_only=true
    restart: always
    ports:
      - "8080:8080"
    volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro

  grafana:
    image: grafana/grafana:3.1.0
    restart: always
    links:
      - influxdb:influxdb
    ports:
      - "3000:3000"
    environment:
      - HTTP_USER=admin
      - HTTP_PASS=admin
      - INFLUXDB_HOST=influxdb
      - INFLUXDB_PORT=8086
      - INFLUXDB_NAME=cadvisor
      - INFLUXDB_USER=root
      - INFLUXDB_PASS=root
```


单独安装cAdvisor

```
docker run \
  --volume=/:/rootfs:ro \
  --volume=/var/run:/var/run:rw \
  --volume=/sys:/sys:ro \
  --volume=/var/lib/docker/:/var/lib/docker:ro \
  --publish=10091:8080 \
  --detach=true \
  --name=cadvisor \
  google/cadvisor:v0.23.2 \
  -storage_driver influxdb \
  -storage_driver_host 192.168.1.206:8086 \
  -storage_driver_db groot_cadvisor \
  -docker_only true
```


[原文链接](https://www.brianchristner.io/how-to-setup-docker-monitoring/)