--- 
layout: post
title: "CentOS Memcache安装配置教程（PHP与Memcache快速搭建）"
comments: true
categories:
 - linux
 - php
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

<font color="RED">注：网上大部分资料均是人云亦云要求yum使用RPMForge源。但经过VPS管理百科实际测试，此源里不包含memcached包，因此无法正常安装。按照VPS管理百科提供的源与方法安装即可。</font>

<STRONG>2、查看已经安装的源</STRONG>
<!--more-->
```vim
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
```
能够找到epel包，说明安装成功。

<STRONG>3、yum安装Memcache服务器与php扩展</STRONG>
<!--more-->
```bash
[root@www ~]# yum install memcached php-pecl-memcache
```
此时应该能正常安装这两个包，而不出现无法找到的情况。

<STRONG>4、安装成功后，检测php是否正常加载了memcache模块：</STRONG>
<!--more-->
<pre class="brush: text" line="1">
[root@www ~]# php -m|grep memcache
memcache
</pre>
返回了“memcache”表示已经安装。

<STRONG>5、设置memcached服务开机自动启动</STRONG>
<!--more-->
<pre class="brush: text" line="1">
[root@www ~]#&nbsp;chkconfig --level 2345 memcached on
</pre>

<STRONG>6、启动memcached服务并重启nginx服务</STRONG>
<!--more-->
<pre class="brush: text" line="1">
[root@www ~]# service memcached start
启动 memcached：[确定]
[root@www ~]# service nginx restart
停止 nginx：[确定]
启动 nginx：[确定]
[root@www ~]# service fastcgi restart
</pre>

注意：php里的fastcgi服务其实就是php-fpm
如果以上重启fastcgi方式无效的话可以试一下以下办法：
<!--more-->
<pre class="brush: text" line="1">
php 5.3.3 下的php-fpm 不再支持 php-fpm 以前具有的 /usr/local/php/sbin/php-fpm (start|stop|reload)等命令，需要使用信号控制：
master进程可以理解以下信号
INT, TERM 立刻终止
QUIT 平滑终止
USR1 重新打开日志文件
USR2 平滑重载所有worker进程并重新载入配置和二进制模块
示例：
php-fpm 关闭：
kill -INT `cat /var/run/php-fpm/php-fpm.pid`
php-fpm 重启：
kill -USR2 `cat /var/run/php-fpm/php-fpm.pid`
查看php-fpm进程数：
ps aux | grep -c php-fpm
</pre>

<STRONG>7、测试php支持memcache是否正常</STRONG>
在apache的网站根目录建立 memcache.php 文件
<!--more-->
<pre class="brush: text" line="1">
vi memcache.php
</pre>
内容如下：
```php
<?php
$memcache = new Memcache();
$memcache->connect('127.0.0.1', 11211);
$memcache->set('key', 'Memcache test successful!', 0, 60);
$result = $memcache->get('key');
unset($memcache);
echo $result;
?>
```
如果一切正常，访问此页面，应该正常返回“Memcache test successful”，至此，Memcached与php扩展memcache安装成功。

Memcached的默认端口为11211，因此在php中使用此端口即可。下面顺便给出个清除memcache所有缓存内容的方法：

执行：
<!--more-->
<pre class="brush: text" line="1">
[root@www ~]# nc localhost 11211
</pre>

然后输入：
<!--more-->
<pre class="brush: text" line="1">
flush_all
quit
</pre>
