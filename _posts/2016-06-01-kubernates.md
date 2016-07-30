--- 
layout: post
title: "持续更新：Kubernates安装与过程中碰到的问题记录"
wordpress_id: 14
wordpress_url: http://www.yunjing.me/?p=14
date: 2016-06-01 14:28:00 +08:00
category: kubernates
tags: 
 - centos
 - docker
 - kubernates
---

本文基于[官方multinode安装手册](http://kubernetes.io/docs/getting-started-guides/docker-multinode/master/)进行操作，将会在两台CentOS 7虚拟机上安装k8s服务框架。

将会安装Kubernetes Master结点、k8s worker结点及SkyDNS服务。

## 安装Kubernetes v1.2.6版本过程问题记录

在启动worker服务时，可能会报以下错误：
```
[1.11.1] Docker bootstrap: Cannot start a container because of oci runtime error.
```

只要新增一个docker deamon的启动参数，重新启动一下flannel对应的docker deamon即可。

```
sh -c 'docker daemon -H unix:///var/run/docker-bootstrap.sock -p /var/run/docker-bootstrap.pid --iptables=false --ip-masq=false --bridge=none --exec-root=/var/run/docker-bootstrap --graph=/var/lib/docker-bootstrap 2> /var/log/docker-bootstrap.log 1> /dev/null &'
```

---------------------------------------------------

在启动SkyDNS时会出现运行一段时间后自动崩溃，原因未知。
经过排查后，在kube2sky服务结点的日志里发现线索：
```
[root@localhost ~]# docker logs a8ae16ed9a92
I0718 12:12:50.201134       1 kube2sky.go:436] Etcd server found: http://127.0.0.1:4001
I0718 12:12:51.204401       1 kube2sky.go:503] Using https://10.0.0.1:443 for kubernetes master
I0718 12:12:51.204446       1 kube2sky.go:504] Using kubernetes API <nil>
```
根据这个"Using kubernetes API <nil>"及其上一句的输出，大概可以知道这个服务结点运行其实是存在问题的，实际上Kubernetes API server并没有部署SSL服务，理论上这个应该是连不上的。

为了验证此理论是否正确，进入kube2sky的容器内执行命令来测试一下：
```
[root@localhost ~]# docker exec -it a8ae16ed9a92 sh
/ # curl https://10.0.0.1:443
sh: curl: not found
/ # wget https://10.0.0.1:443
Connecting to 10.0.0.1:443 (10.0.0.1:443)
wget: can't connect to remote host (10.0.0.1): Connection timed out
```

参与GitHub上的[Issue:22684](https://github.com/docker/docker/issues/22684)，大概解决办法在skydns.yml里给`kube2sky`新增一个启动参数：
```
# command = "/kube2sky"
- --domain=cluster.local
- --kube_master_url=http://<your-master-ip>:8080
```
---------------------------------------------------

## 安装Kubernetes v1.3.3版本过程问题记录
