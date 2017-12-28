--- 
layout: post
title: "使用KCP协议实现HTTP代理服务"
comments: true
categories:
 - linux
 - kcp
 - http
---

前言
本文演示如何使用KCP协议实现HTTP代理服务，用于国内服务端进程无法连接Facebook开放平台进行用户身份授权验证的功能。

运行环境
CentOS 7.0

步骤
1、创建IAM的Group、User等；包括EC2全部权限、S3全部权限、IAM全部权限；
2、在安装机进行aws configure指定IAM权限；
3、安装Kubernates环境；
