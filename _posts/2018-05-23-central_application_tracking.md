---
layout: post
title: 分布式实时监控系统（不断更新）
comments: true
excerpt: true
categories:
 - 监控
---

在2010年, Google发表了一篇名为"Dapper, a Large-Scale Distributed Systems Tracing Infrastructure"的论文，在论文中介绍了Google在生产环境中大规模分布式系统下的跟踪系统Dapper的设计和使用经验。
实时监控告警和应用性能分析诊断平台，主要包括应用端监控和移动端监控等。

## 为什么需要跟踪系统

### 故障快速定位
快速的故障定位非常重要，一个良好设计的系统需要提供快速检测、隔离及修复问题的机制，而快速定位则是这个机制中的重要一环，由于缺乏快速定位的机制，当关键业务出现服务停止或者性能恶化，对业务有直接影响时，由于缺乏有效和快速的应对措施，往往由最终用户的反馈作为整个定位的开始一环，而此时已经是损失很难挽回的时间点了。

### 恶性循环的形成
由于缺乏快速定位机制，那些长期存在的性能低下问题以及重复出现的问题很难精确定位，产生了大量的技术债务。而随着新的功能的不断添加，同样无法尽早或者预先发现问题导致问题也不断的累积。修改自身也可能会导致更多问题的引入，从而形成了恶性循环。

### 现状把握&决策支持
根据跟踪系统提供的数据可以提供及时和全面的性能报告，并且在此基础之上，能够做进一步的分析从而提供决策支持的所需数据，比如根据收集的信息可以对用户行为/模式分析：从而进一步地进行测量和管控。

## 启示1：跟踪系统的两个重要需求
在Dapper的论文这样认为，有两个重要的需求需要予以满足：ubiquitous deployment和continuous monitoring。而这两个需求分别是对跟踪系统的作用范围（无死角跟踪）和运行方式（24小时不间断）做出了规范。

## 启示2：跟踪系统的三大设计要点

### 低损耗（Low overhead）
跟踪系统的接入不应该产生很高的性能损耗。具体要求多低，Dapper论文中使用的是negligible(微不足道的，可以忽略的), 如果从度量的角度来讲，pinpoint的实现是3%一下。

### 应用级透明（Application-level transparency）
由于业务逻辑的不断增长，如果跟踪系统在设计上无法做到对于应用程序的较小侵入性，会导致大量的维护成本以及引入额外缺陷的机率大大提高。理想的做法是能达到论文中所提到的那样：程序员应该不需要意识到有追踪系统的存在。而在实际的实践中，比如引入zipkin或者pinpoint，或多或少还是需要一定的成本，至少跟踪系统的埋点不与业务逻辑产生紧密耦合还是可以实现的，但是完全无意识追踪系统的存在，在具体的实现上还是需要结合系统特点，不是一蹴而就的目标。

### 扩展性（Scalability）
至少跟踪系统应该要能适应未来几年的服务和集群的扩展。

## 启示3：跟踪系统的KPI
* 采样率：根据Google的实践经验，采样率效率1/16时便能避免明显延迟，性能损耗在实验误差范围之内。跟踪大量服务时，采样率调整到1/1024就有足够的跟踪数据，同时能保证性能损耗非常低。Dapper的第一个生产版本对所有进程采用1/1024的统一采样率，后来演进为可变采样率。
* 损耗：跟踪系统自身的性能影响对于系统应该是能够忽略不计的，pinpoint控制在3%之内。
* 实效性：跟踪数据产生之后，分析的速度需要及时快速，理想状况数据在一分钟之内能够统计出来，以便对于生产环境的异常状况作出快速反应。

## 启示5：准确定位延迟问题
当一个系统涉及到多个子系统和多个开发团队的情况下，端到端的性能问题的根本原因定位可以通过跟踪系统来进行。Google通过使用Dapper描述了这种情况下的必需数据，其实践有如下经验：
* 问题往往来源于一些意想不到的服务之间的交互，问题的纠正往往比较容易，而在Dapper引入之前发现较为困难
* 简化跟踪的接口，使用唯一的追踪ID，便于集成和检测

## 启示8：Dapper系统的核心构成
Dapper是通过trace tree和span构建跟踪系统的。

### span
span是用用于记录一个服务调用的过程的结构，一个典型的跟踪系统中，一次RPC调用会对应到一个的span上，dapper中定义了span相关的如下信息：

![](/files/2018/06/20180611204738876.png "A detailed view of a single span")

* span名称：用于记录span的名称
* spanid：用于记录span的Id，一般用全局唯一的64位整数表示
* 父spanid：父span的spanid，用于描述跟踪树结构
* 事件信息：cs/cr/sr/ss四种事件类型，span不同的事件类型对应不同的时间戳，根据这些时间戳，可计算出不同阶段的耗时信息。
* annotation：一般，事件信息在annotation中存放，另外自定义的信息也可以与之关联

|缩写|全称|说明|
| -- | -- | - |
| cs | client send | 客户端/消费者发起请求 |
| cr | client receive | 客户端/消费者接收到应答 |
| sr | server receive | 服务端/生产者接收到请求 |
| ss | server send | 服务端/生产者发送应答 |

### trace tree

一个请求可能跟多个服务调用关联，每次服务的调用与一个span进行关联，而span之间通过父spanid进行连接，这样所组成的一个树形结构就是所谓的跟踪树，这个结构显示体现了某一请求的服务调用链的状况。比如论文中的例子即为一个典型的三层架构的例子：具体说明如下： 

![](/files/2018/06/20180611204839950.png "The causal and temporal relationships between five spans in a Dapper trace tree.")

| 层次 | 服务名称 | 父span |	调用顺序 |
| ---- | --------| ----- | --------- |
| 前端 |	Frontend：A |	无 |	1 |
| 中间 |	MiddleTier：B |	A |	2 |
| 中间 |	MiddleTier：C |	A |	3 |
| 后端 |	Backend：D |	C |	4 |
| 后端 |	Backend：E | C |	5 |

![](/files/2018/06/20180611204853221.png "The path taken through a simple serving system on behalf of user request X. The letter-labeled nodes represent processes in a distributed system")

这样的一个树形结构，表现出来的调用顺序则是：A->B->C->D->E。这样的一个简洁的设计，就是dapper的主要构成，可以看出其与业务逻辑基本不相关的一个通用的模型，而在zipkin等的实现中，可以清晰地看到Dapper的整体设计思路。

## 启示9：定位全局网络流量和使用率
Dapper不是设计用来做链路级的监控的，但是在实践中发现，其比较适合去做集群之间活动性的应用及行为分析，显示集群之间最活跃的网络流量的应用级热点，与传统的网络相关的工具相比，最重要的特点是Dapper可以定位到应用程序级别的根本问题。

## 启示10：Dapper不太适合的场景
Dapper的模型的隐含前提是不同的子系统使用同一个被跟踪的请求所产生的连锁链式调用栈的情况。如果是多个追踪请求合并起来，而最终只使用其中的一个的情况则无法很好地对应。
Dapper可以找出性能瓶颈，但是并不一定能准确定位到根本原因，因为定位出很慢的结果往往是由其他请求造成的，而这则需要进一步的分析。
Dapper的设计主要是针对在线服务的系统，尤其是一个用户请求的系统行为，但是离线的情况下，则不能直接使用，需要一些人工的关联和干预行为。

## Dapper的实现
|   名称   |  开发商  | 类型 |   源码地址  |
| -------- | --------| ----- | ---------- |
|  zipkin  | twitter |  开源  | [https://github.com/openzipkin/zipkin](https://github.com/openzipkin/zipkin) |
| pinpoint | naver   |  开源  | [https://github.com/naver/pinpoint](https://github.com/naver/pinpoint) |
| appdash  | sourcegraph | 开源 | [https://github.com/sourcegraph/appdash](https://github.com/sourcegraph/appdash) |
| cat | 大众点评 | 开源 | [https://github.com/dianping/cat](https://github.com/dianping/cat) |
| hydra | 京东 | 开源 | [https://github.com/odenny/hydra](https://github.com/odenny/hydra) |
| 鹰眼 | 阿里巴巴 | 闭源 | - |

## 大众点评的cat实现参考

### 客户端SDK设计

| 用途 | 示例 | 对应接口 |
| --------------- | ------------- | ----- |
| 一段代码执行时间 | url/sql响应时间 | Transaction |
| 一段代码执行次数 | Exception出现次数 | Event |
| 定时执行某些代码 | 分钟粒度CPU,IO | HeartBeat |
| 一个指标的变化值 | 监控销售额 | Metric |

### 消息生命周期
构建、传输 、分析、存储、展示


## 参考链接
[Dapper, a Large-Scale Distributed Systems Tracing Infrastructure](https://ai.google/research/pubs/pub36356)
[开源分布式监控 CAT 系统的高可用实践](http://www.infoq.com/cn/presentations/the-practice-of-open-source-distributed-monitoring-cat-system?utm_source=infoq&utm_medium=videos_homepage&utm_campaign=videos_row2)
[大规模分布式系统的跟踪系统：Dapper设计给我们的启示](https://blog.csdn.net/liumiaocn/article/details/80657661)
