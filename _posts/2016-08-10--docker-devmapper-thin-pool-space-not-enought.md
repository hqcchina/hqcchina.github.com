--- 
layout: post
title: "Docker镜像生成时报错devmapper空间不足"
wordpress_id: 20
wordpress_url: http://www.yunjing.me/?p=20
date: 2016-08-10 10:09:00 +08:00
category: docker
tags: 
 - docker
 - devmapper
---

今天在例行代码CI时出现例外的编译报错，查看了一下日志发现了下面这段：

```
devmapper: Thin Pool has 96867 free data blocks which is less than minimum required 163840 free data blocks. Create more free space in thin pool or use dm.min_free_space option to change behavior
```

上服务器看了一下占用空间：

```
$ sudo du /data/docker/devicemapper/devicemapper/data -h
94G     /data/docker/devicemapper/devicemapper/data
```

Google一番大概意思是device-mapper在删除镜像时没有回收，然后此BUG的修复补丁还没有合并进入Linux内核主分支的计划，需要自行打补丁。  
然而自己并不懂怎么打这个补丁，所以，尝试升级docker及device-mapper试试看。

```
  docker:  1.11.1 -> 1.12.0
  device-mapper: 7:1.02.107-5.el7 -> 7:1.02.107-5.el7_2.5
```

最后还是还有结论，目前只是把一些没用的镜像和容器都删掉腾出了一些空间，然后，重新启动一下docker服务后意外恢复正常。