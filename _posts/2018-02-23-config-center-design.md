---
layout: post
title: 配置中心的设计与选型（未完成）
comments: true
categories:
 - 配置中心
 - Golang
---

<B>此文未完结，待补充完善</B>

## 服务治理

配置中心的配置管理和动态配置下发能力，使得它可以承担服务治理中的Control Plane的职责，它可以把服务治理相关的配置下发给实际的执行模块，比如Dubbo，或者是Spring Cloud集成的Netflix治理工具库，比如Hystrix以及Ribbon。

服务治理的主要功能包括：
* 注册/发现，服务实例启动后，将会自动通过Hawk Client向配置中心注册，运维可以在配置中心门户查看服务的实例列表，并进行实例级别的服务治理。
* 负载均衡，调用其他服务时，可以灵活指定其负载均衡策略，如 round robin、hash、consistent hash、weight based round robin等等。
* 路由，配置微服务的路由策略和路由规则，路由策略如本地优先，本地数据中心优先等等，路由规则如白名单，黑名单，流量引导，读写分离，前后端分离，灰度升级等等。
* 限流，针对某个调用方限流对象进行QPS限流，分钟级或秒级，或针对所有微服务客户端进行限流。
* 降级，针对某个降级对象，进行手动或者自动降级（容错），并制定降级策略，抛出异常或特定返回值。
* 容错，针对某个微服务，设定调用超时与重试策略。
* 熔断，针对某个微服务或者RPC方法，设定熔断的触发条件，可以强制熔断，或者自动熔断，也可以取消熔断，运维可以指定熔断参数，比如异常，错误码，失败次数比率，时间窗口，以及窗口请求书等等。

## 参考资料
* [分布式配置中心架构与实战](http://dockone.io/article/2992)
