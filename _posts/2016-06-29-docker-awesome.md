--- 
layout: post
title: "Docker常用命令收藏"
wordpress_id: 17
wordpress_url: http://www.yunjing.me/?p=17
date: 2016-06-29 15:40:00 +08:00
category: docker
tags: 
 - docker
---

移除所有镜像 
$ docker ps -q -a | xargs docker rm 

移除所有none名称的镜像 

```shell
$ docker rmi $(docker images | grep "^<none>" | awk '{print $3}') 
```
