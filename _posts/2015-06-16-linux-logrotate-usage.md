--- 
layout: post
title: "被遗忘的logrotate"
comments: true
categories:
 - linux
 - logrotate
---
我发现很多人的服务器上都运行着一些诸如每天切分Nginx日志之类的CRON脚本，大家似乎遗忘了logrotate，争先发明自己的轮子，这真是让人沮丧啊！就好比明明身边躺着现成的性感美女，大家却忙着自娱自乐，罪过！

## logrotate介绍
logrorate程序是一个日志文件管理工具，可以用来分割日志文件，删除旧的日志文件，并创建新的日志文件，起到“转储”作用。配置CRON更可以做按日期分割整个日志文件为各个小文件！！！

logrorate是基于CRON来运行的，其脚本是「/etc/cron.daily/logrotate」：
```bash
#!/bin/sh

/usr/sbin/logrotate /etc/logrotate.conf
EXITVALUE=$?
if [ $EXITVALUE != 0 ]; then
    /usr/bin/logger -t logrotate "ALERT exited abnormally with [$EXITVALUE]"
fi
exit 0
```

实际运行时，logrotate会调用配置文件「/etc/logrotate.conf」：
```vim
# see "man logrotate" for details
# rotate log files weekly
weekly

# keep 4 weeks worth of backlogs
rotate 4

# create new (empty) log files after rotating old ones
create

# uncomment this if you want your log files compressed
#compress

# RPM packages drop log rotation information into this directory
include /etc/logrotate.d

# no packages own wtmp -- we'll rotate them here
/var/log/wtmp {
    monthly
    minsize 1M
    create 0664 root utmp
    rotate 1
}

# system-specific logs may be also be configured here.
```

简单说明：  
weekly : 所有的日志文件每周转储一次，也可以是每日daily或每月monthly；  
rotate 4 : 表示保留4份；  
compress : 表示压缩每一份日志文件，默认使用gzip压缩；  
notifempty : 如果是空文件的话，不转储；  
postrotate/endscript : 日志转储后执行的脚本；  
create : 自动创建新的日志文件；  
missingok : 表示如果找不到log也不用报错；  
delaycompress : 表示延后压缩直至下一次rorate；  
copytruncate : 表示先复制日志内容后再清空，与之相对是create方法；  
dateext : 表示以日期作为后缀，默认以数字1、2、3等作为后缀；  

这里的设置可以理解为logrotate的缺省值，当然了，可以我们在「/etc/logrotate.d」目录里放置自己的配置文件，用来覆盖logrotate的缺省值。

## logrotate演示
按天保存一周的Nginx日志压缩文件，配置文件为「/etc/logrotate.d/nginx」：
```vim
/usr/local/nginx/logs/*.log {
    daily
    dateext
    compress
    rotate 7
    sharedscripts
    postrotate
        kill -USR1 `cat /var/run/nginx.pid`
    endscript
}
```
如果你等不及CRON，可以通过如下命令来手动执行：
`# logrotate -f /etc/logrotate.d/nginx`
当然，正式执行前最好通过Debug选项来验证一下，这对调试也很重要：
`# logrotate -d -f /etc/logrotate.d/nginx`

BTW：类似的还有Verbose选项，这里就不多说了。

logrotate命令格式：
```bash
logrotate [OPTION...] <configfile>
-d, --debug ：debug模式，测试配置文件是否有错误。
-f, --force ：强制转储文件。
-m, --mail=command ：发送日志到指定邮箱。
-s, --state=statefile ：使用指定的状态文件。
-v, --verbose ：显示转储过程。
```

## logrotate问答
### 问题：sharedscripts的作用是什么？

大家可能注意到了，我在前面Nginx的例子里声明日志文件的时候用了星号通配符，也就是说这里可能涉及多个日志文件，比如：access.log和error.log。说到这里大家或许就明白了，sharedscripts的作用是在所有的日志文件都轮转完毕后统一执行一次脚本。如果没有配置这条指令，那么每个日志文件轮转完毕后都会执行一次脚本。

### 问题：rotate和maxage的区别是什么？

它们都是用来控制保存多少日志文件的，区别在于rotate是以个数为单位的，而maxage是以天数为单位的。如果我们是以按天来轮转日志，那么二者的差别就不大了。

### 问题：为什么生成日志的时间是凌晨四五点？

前面我们说过，Logrotate是基于CRON运行的，所以这个时间是由CRON控制的，具体可以查询CRON的配置文件「/etc/crontab」，可以手动改成如23:59等时间执行：
```vim
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# run-parts
01 * * * * root run-parts /etc/cron.hourly
59 23 * * * root run-parts /etc/cron.daily
22 4 * * 0 root run-parts /etc/cron.weekly
42 4 1 * * root run-parts /etc/cron.monthly
```
如果使用的是新版CentOS，那么配置文件为：`/etc/anacrontab`。

### 问题：如何告诉应用程序重新打开日志文件？

以Nginx为例，是通过postrotate指令发送USR1信号来通知Nginx重新打开日志文件的。但是其他的应用程序不一定遵循这样的约定，比如说MySQL是通过flush-logs来重新打开日志文件的。更有甚者，有些应用程序就压根没有提供类似的方法，此时如果想重新打开日志文件，就必须重启服务，但为了高可用性，这往往不能接受。还好Logrotate提供了一个名为copytruncate的指令，此方法采用的是先拷贝再清空的方式，整个过程中日志文件的操作句柄没有发生改变，所以不需要通知应用程序重新打开日志文件，但是需要注意的是，在拷贝和清空之间有一个时间差，所以可能会丢失部分日志数据。

BTW：MySQL本身在support-files目录已经包含了一个名为mysql-log-rotate的脚本，不过它比较简单，更详细的日志轮转详见[「Rotating MySQL Slow Logs Safely」](https://www.percona.com/blog/2013/04/18/rotating-mysql-slow-logs-safely/)。

…

熟悉Apache的朋友可能会记得cronolog，不过Nginx并不支持它，有人通过mkfifo命令曲线救国，先给日志文件创建管道，再搭配cronolog轮转，虽然理论上没有问题，但效率上有折扣。另外，Debian/Ubuntu下有一个简化版工具savelog，有兴趣可以看看。
