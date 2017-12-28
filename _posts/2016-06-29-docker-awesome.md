--- 
layout: post
title: "Docker常用命令收藏"
comments: true
categories:
 - docker
---

移除所有镜像 
$ docker ps -q -a | xargs docker rm 

移除所有none名称的镜像 

```shell
$ docker rmi $(docker images | grep "^<none>" | awk '{print $3}') 
```
