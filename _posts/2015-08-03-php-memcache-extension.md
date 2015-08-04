--- 
layout: post
title: "CentOS Memcache安装配置教程（PHP与Memcache快速搭建）"
wordpress_id: 7
wordpress_url: http://www.yunjing.me/?p=7
date: 2015-08-03 20:24:00 +08:00
category: jquery
tags: 
 - php
 - extension
 - memcache
---
本教程将介绍如何在CentOS中安装Memcache缓存服务。Memcache是一个与php兼容的内存高速缓存插件，不仅可以缓存变量等对象，而且可以与MySQL配合，缓存数据查询。由于Memcache在内存中缓存数据，因此它的读取写入速度非常之快，能为大容量快速变化的动态数据提供高速缓存。

由于编译安装Memcache步骤相对复杂一些，因此本文将以CentOS系统下yum直接安装Memcache为例进行讲解，这种安装方法快捷简便。

<STRONG>1、由于CentOS系统默认源没有memcache安装包，因此需要导入第三方的源。执行如下两条命令：</STRONG>
<!--more-->
<pre class="brush: text" line="1">
[root@www ~]#&nbsp;wget http://soft.bootf.com/rpm/epel-release-5-4.noarch.rpm
[root@www ~]#&nbsp;rpm -ivh epel-release-5-4.noarch.rpm
</pre>

<COLOR="RED">注：网上大部分资料均是人云亦云要求yum使用RPMForge源。但经过VPS管理百科实际测试，此源里不包含memcached包，因此无法正常安装。按照VPS管理百科提供的源与方法安装即可。</COLOR>

<STRONG>2、查看已经安装的源</STRONG>
<!--more-->
<pre class="brush: text" line="1">
[root@www ~]# yum repolist
Loaded plugins: fastestmirror
Loading mirror speeds from cached hostfile
* base: centos.ustc.edu.cn
* epel: mirrors.ustc.edu.cn
* extras: centos.ustc.edu.cn
* rpmforge: fr2.rpmfind.net
* updates: centos.ustc.edu.cn
repo id repo name status
base CentOS-5 - Base 2,705
epel Extra Packages for Enterprise Linux 5 - i386 5,579
extras CentOS-5 - Extras 282
updates CentOS-5 - Updates 455
repolist: 20,115
</pre>
能够找到epel包，说明安装成功。
