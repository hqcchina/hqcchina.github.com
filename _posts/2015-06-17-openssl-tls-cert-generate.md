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

## 自签名证书生成思路
在自签名证书生成的整个过程中，会涉及及生成很多个证书文件，简单点就是先生成一组证书用以标识CA机构合法的，一般会包括一个公钥和一个私钥，然后用这个CA机构的私钥去生成一个客户端的CA证书，并发放给客户端使用。


## 自签名证书生成方法

### 方法一、通过OpenSSL生成
<!--more-->
<pre class="brush: text" line="1">
查看OpenSSL版本
$ openssl version -a


生成RSA密钥的方法
$ openssl genrsa -des3 -out server.key 2048
这个命令会生成一个2048位的密钥，同时有一个des3方法加密的密码，如果你不想要每次都输入密码，可以改成：
$ openssl genrsa -out server.key 2048
建议用2048位密钥，少于此可能会不安全或很快将不安全。


生成一个证书请求
$ openssl req –new -x509 -key server.key -out server.pem
这个命令将会生成一个证书请求，当然，用到了前面生成的密钥server.key文件
这里将生成一个新的文件server.pem，即一个证书请求文件，你可以拿着这个文件去数字证书颁发机构（即CA）申请一个数字证书。CA会给你一个新的文件ca.pem，那才是你的数字证书。


如果是自己做测试，那么证书的申请机构和颁发机构都是自己。就可以用下面这个命令来生成证书：
$ openssl req -new -x509 -key server.key -out ca.pem -days 1095
这个命令将用上面生成的密钥server.key生成一个数字证书ca.pem
</pre>

### 方法二、通过Go自带的代码文件生成
在Go的安装目录下有一个文件$GO_ROOT/src/crypto/tls/generate_cert.go，可以使用这个代码文件生成CA机构的一对公私钥证书，然后再去CA机构或者自签名生成CA证书。

查询Go的安装目录可以使用以下命令：
$ whereis go

### 方法三、通过GitHub上网友提供的库生成
使用Go语言编写的一个开源库，代码只有一个文件，地址如下：
https://github.com/deckarep/EasyCert
用法也非常简单，go run EasyCert.go -ca=China -host=192.168.1.51即可。
运行结束后会自动生成三个文件，分别是一对CA机构的公私钥以及一个CA证书。

## 结论


## 参考资料
<a href="http://rhythm-zju.blog.163.com/blog/static/310042008015115718637/">基于 OpenSSL 的 CA 建立及证书签发 </a>

<a href="http://segmentfault.com/a/1190000002569859">基于OpenSSL自建CA和颁发SSL证书</a>

<a href="http://blog.csdn.net/php_boy/article/details/6660697">Linux下CA证书生成</a>