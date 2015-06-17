--- 
layout: post
title: "gRPC库的TLS自签名证书生成"
wordpress_id: 5
wordpress_url: http://www.yunjing.me/?p=5
date: 2015-06-17 13:38:00 +08:00
category: tls
tags: 
 - tls
 - openssl
 - golang
 - grpc
 - ca
---
Google出品的gRPC问世有一段时间，想着抽个空来做个DEMO上个手看看效果如何。
服务端使用grpc-go库，客户端使用grpc-objectivec版本并运行在iPhone设备上。
除了开发过程中，grpc库更新仍然很频繁外，同时伴随偶尔的库升级之后无法编译外，一路上还是很阵痛的。
核心功能上终于没有太大的问题了，想着通信加密来一个SSL/TLS加密，就开始探索之路，至今还在坑中！！！

由于软妹币不够，所以，思索着自己生成自签名证书，但是在iPhone模拟器上运行时，客户端的CA证书总是报各式各样的问题，解决一个又有另一个，像是一直没有摸到头绪的感觉，下面记录一下目前探索到的各个生成证书的办法。

### 方法一、通过OpenSSL生成

