--- 
layout: post
title: "持续更新：Kubernates安装与过程中碰到的问题记录"
comments: true
categories:
 - centos
 - docker
 - kubernates
---

服务器：
1、CentOS7: 10.10.1.70 `master`  
2、CentOS7: 10.10.1.73  

运行环境：
1、`uname -a` : Linux localhost.localdomain 3.10.0-229.20.1.el7.x86_64 #1 SMP Tue Nov 3 19:10:07 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux  
2、`docker -v`: Docker version 1.11.2, build b9f10c9  

将会安装以下服务或组件：
1、k8s master  
2、k8s worker  
3、SkyDNS  
4、dashboard  

安装手册：
适用于v1.2.6版本：[Portable Multi-Node Clusters](http://kubernetes.io/docs/getting-started-guides/docker-multinode/master/)
适用于v1.3.4版本：[Portable Multi-Node Clusters](https://github.com/kubernetes/kube-deploy/tree/master/docker-multinode)

## 安装Kubernetes v1.2.6版本过程问题记录

1、无法拉取gcr.io镜像被墙问题  
使用squid3在海外服务器搭建HTTP代理，然后，配置docker服务使用HTTP代理。

```
$ sudo tee /etc/systemd/system/docker.service.d/http-proxy.conf <<-'EOF'
[Service]
Environment="HTTP_PROXY=http://<your-squid3-ip>:<port>/"
Environment="HTTPS_PROXY=http://<your-squid3-ip>:<port>/"
Environment="NO_PROXY=localhost,127.0.0.1,<your-private-docker-registry>"
EOF
```

修改之后，重启docker服务即可：

```
$ sudo systemctl daemon-reload
$ sudo systemctl restart docker
```

2、启动worker结点上的docker服务报"Cannot start a container because of oci runtime error."  
在启动worker服务时，可能会报以下错误：

```
[1.11.1] Docker bootstrap: Cannot start a container because of oci runtime error.
```

只要新增一个docker deamon的启动参数，重新启动一下flannel对应的docker deamon即可。

```shell
sh -c 'docker daemon -H unix:///var/run/docker-bootstrap.sock -p /var/run/docker-bootstrap.pid --iptables=false --ip-masq=false --bridge=none --exec-root=/var/run/docker-bootstrap --graph=/var/lib/docker-bootstrap 2> /var/log/docker-bootstrap.log 1> /dev/null &'
```

3、SkyDNS服务启动不正常，出现"Using Kubernetes API<nil>"报错  
在启动SkyDNS时会出现运行一段时间后自动崩溃，原因未知。
经过排查后，在kube2sky服务结点的日志里发现线索：

```shell
[root@localhost ~]# docker logs a8ae16ed9a92
I0718 12:12:50.201134       1 kube2sky.go:436] Etcd server found: http://127.0.0.1:4001
I0718 12:12:51.204401       1 kube2sky.go:503] Using https://10.0.0.1:443 for kubernetes master
I0718 12:12:51.204446       1 kube2sky.go:504] Using kubernetes API <nil>
```

根据这个"Using kubernetes API <nil>"及其上一句的输出，大概可以知道这个服务结点运行其实是存在问题的，实际上Kubernetes API server并没有部署SSL服务，理论上这个应该是连不上的。

为了验证此理论是否正确，进入kube2sky的容器内执行命令来测试一下：

```shell
[root@localhost ~]# docker exec -it a8ae16ed9a92 sh
/ # curl https://10.0.0.1:443
sh: curl: not found
/ # wget https://10.0.0.1:443
Connecting to 10.0.0.1:443 (10.0.0.1:443)
wget: can't connect to remote host (10.0.0.1): Connection timed out
```

参考GitHub上的[docker issue:22684](https://github.com/docker/docker/issues/22684)，大概原因是由于k8s的master url是错误的，简单的解决办法是给其指定正确的master地址，如在skydns.yml里给`kube2sky`新增一个启动参数：

```
# command = "/kube2sky"
- --domain=cluster.local
- --kube_master_url=http://<your-master-ip>:8080
```

## 安装Kubernetes v1.3.3版本过程问题记录

1、启动master结点的apiserver时报"Invalid Authentication Config: open /srv/kubernetes/basic_auth.csv: no such file or directory"错误  
报错信息如下：

```shell
[root@localhost kubernetes]# docker logs -f b1afbd49ea94
I0730 03:58:27.673007       1 genericapiserver.go:606] Will report 42.62.46.70 as public IP address.
W0730 03:58:27.678801       1 server.go:173] No RSA key provided, service account token authentication disabled
F0730 03:58:27.678856       1 server.go:206] Invalid Authentication Config: open /srv/kubernetes/basic_auth.csv: no such file or directory
```

Google了一圈之后，没有发现有效的解决办法。无奈自行重启docker服务，把当前已有容器全部移除，然后，直接启动k8s一切正常。

2、k8s自动启动dashboard报"dial tcp 10.0.0.1:443: getsockopt: no route to host"  
这和1.2.6版本时出现的类似问题是一样一样的，解决办法无非就是指定正确的master地址。但，此次由于这是k8s自行创建的，需要去探索在哪里可以修改这个master地址，或者，直接把api server监听整成SSL的。

由于此办法无效，转而使用其他比较新的文档进行重新安装。见上文提到的另一种Multi Node安装手册。

然后发现此问题仍然存在，据说是pod的创建时间早于secret的创建时间，解决办法如下：

```
$ kubectl get secret --namespace=kube-system
$ kubectl delete secret <secret-name> --namepace=kube-system
$ kubectl delete po <po-name> --namespace=kube-system
```

## 安装Kubernetes v1.4.x版本过程问题记录

从1.4.x版本开始，kubernetes使用一种叫kubeadm的工具，CentOS7下面直接使用yum安装即可。

体验了一下这个yum安装，然而，无法使用正常启动，等待新版本发布。

1、启动时报hostname不合法

```
# kubeadm init
Running pre-flight checks
preflight check errors:
        hostname "vm_208_92_centos" must match the regex [a-z0-9]([-a-z0-9]*[a-z0-9])?(\.[a-z0-9]([-a-z0-9]*[a-z0-9])?)* (e.g. 'example.com')
```

尝试把 /etc/hostname 文件里的内容修改为 k8s.crazyant.com 尝试了一下

