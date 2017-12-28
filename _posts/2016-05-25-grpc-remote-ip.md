--- 
layout: post
title: "grpc-go里获取远程连接客户端的IP方法"
comments: true
categories:
 - grpc
 - golang
---

在刚刚开机后，一直开机启动的Shadowsocks突然报“端口已被占用”错误，报错信息如下：

```
[2016-05-24 13:44:31] System.Exception: 端口已被占用
   在 Shadowsocks.Controller.Listener.Start(Configuration config)
   在 Shadowsocks.Controller.ShadowsocksController.Reload()
```

大概原因可能是本机代理的1080端口被其他进程占用：

```
netstat -aon|findstr "1080"
```

显示信息如下：

```
C:\Users\yunjing>netstat -aon|findstr "1080"
  TCP    0.0.0.0:1080           0.0.0.0:0              LISTENING       4492
  TCP    127.0.0.1:1080         127.0.0.1:1382         ESTABLISHED     4492
  TCP    127.0.0.1:1080         127.0.0.1:1590         ESTABLISHED     4492
  TCP    127.0.0.1:1588         127.0.0.1:1080         ESTABLISHED     6572
  TCP    127.0.0.1:1590         127.0.0.1:1080         ESTABLISHED     4972
  TCP    127.0.0.1:1598         127.0.0.1:1080         ESTABLISHED     6572
  TCP    127.0.0.1:1601         127.0.0.1:1080         ESTABLISHED     4972
  TCP    127.0.0.1:1655         127.0.0.1:1080         TIME_WAIT       0
  TCP    127.0.0.1:1723         127.0.0.1:1080         ESTABLISHED     6572
  TCP    127.0.0.1:1728         127.0.0.1:1080         ESTABLISHED     4972
  TCP    127.0.0.1:1765         127.0.0.1:1080         ESTABLISHED     6572
```

大概就是进程4492、6572、4972之类的导致的，
打开进程管理器把4492进程干掉

![按进程号Kill进程示例](/files/2016/05/process-manager.png)

重新打开Shadowsocks一切正常。
