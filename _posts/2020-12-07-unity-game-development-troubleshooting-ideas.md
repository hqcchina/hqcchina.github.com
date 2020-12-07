---
layout: post
title: Unity游戏开发问题排查思路
comments: true
categories:
 - unity
---

* 服务器
  * 磁盘
    * 利用 `df -h` 获取磁盘空间状态
    * 利用 `ls -lh` 查看文件大小
    * 利用 `du -h` 查看文本夹大小
  * CPU
    * 利用top查看进程状态
      * 获取CPU/内存使用率最高的进程pid、启动命令等信息
    * 利用 `top -H` 查看线程状态
      * 获取CPU/内存使用率最高的线程信息
  * 内存
    * 利用 `free -h` 查看内存使用情况
* 数据库
  * 慢SQL
    * 利用explain执行计划优化SQL
      * 利用索引提速
      * 小表驱动大表
    * 利用 `show processlist` 然后kill终结慢查询
  * 连接过多
    * 利用 `set global max_connections`增大连接数
    * 利用 `show processlist` 然后kill终结多余的连接
  * 死锁
    * 事务隔离级别
    * 锁时序分析
      * 表锁
      * 页锁
      * 行锁
        * Record Lock
        * Gap Lock
        * Next-Key Lock
      * 意向锁
* Redis
  * 内存不足
    * 大key
      * rdbtools查询大key
      * 利用redis-cli查询大key
      * 利用debug object key查看key序列化后的大小
    * config set maxmemory临时增加内存
    * 指定内存淘汰机制
  * 连接数过多
    * config set maxclient临时增加最大连接数
    * 限制客户端最大连接数
  * 慢命令
    * config set slowlog-log-lower-than设置慢命令阙值
    * config set slowlog-max-len设置最大慢命令记录保存数
    * 利用slowlog get查询慢命令
  * 查询网络延迟
    * redis-cli -latency查询延迟信息
* 网络
  * 利用 `netstat` 查询统计网络状态
* 业务线
