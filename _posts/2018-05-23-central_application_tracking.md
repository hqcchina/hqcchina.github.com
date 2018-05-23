---
layout: post
title: 分布式实时监控系统（不断更新）
comments: true
excerpt: true
categories:
 - 监控
---

实时监控告警和应用性能分析诊断平台，主要包括应用端监控和移动端监控等

## 客户端SDK设计

|: 用途 :|: 示例 :|: 对应接口 :|
| 一段代码执行时间 | url/sql响应时间 | Transaction |
| 一段代码执行次数 | Exception出现次数 | Event |
| 定时执行某些代码 | 分钟粒度CPU,IO | HeartBeat |
| 一个指标的变化值 | 监控销售额 | Metric |

## 消息生命周期
### 构建

### 传输 

### 分析 

### 存储

### 展示

## 参考链接
[开源分布式监控 CAT 系统的高可用实践](http://www.infoq.com/cn/presentations/the-practice-of-open-source-distributed-monitoring-cat-system?utm_source=infoq&utm_medium=videos_homepage&utm_campaign=videos_row2)
