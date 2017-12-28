--- 
layout: post
title: "Docker下的跨主机网络解决简单方案"
wordpress_id: 15
wordpress_url: http://www.yunjing.me/?p=15
date: 2016-06-02 14:55:00 +08:00
category: docker
tags: 
 - docker
 - vxlan
 - linux
---

前言
本文演示如何解决跨主机间的Docker相互访问问题，Docker官方采用的overlay的VxLan方案，但是被绑定Swarm，而本文仅仅是用Docker本身。

运行环境
CentOS 7.0

步骤
1、创建IAM的Group、User等；包括EC2全部权限、S3全部权限、IAM全部权限；
2、在安装机进行aws configure指定IAM权限；
3、安装Kubernates环境；

[理解Docker跨多主机容器网络](http://tonybai.com/2016/02/15/understanding-docker-multi-host-networking/)
[Docker+OpenvSwitch搭建VxLAN实验环境](http://www.cnblogs.com/yuuyuu/p/5180827.html)