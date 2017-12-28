--- 
layout: post
title: "HTTP压力测试时大量Timeout的问题解决"
comments: true
categories:
 - test
 - http
---

Because we want to test the scenario that clients keep generating new connections to the server, we specifically disabled keep-alive for the underlying connection. After running the program for a while we found that the client used up the IP port range reserved for client connection which is roughly 28k ports. The result is that on the client host, you observe a lot of TCP connections in TIME_WAIT state. This is an expected behavior of TCP connections. I did the following trick to workaround the problem:

* Tweak the OS configuration on the running host. We run the client on a Linux server, you can follow this article to set net.ipv4.tcp_tw_reuse to 1.
* Change the Dial of http.Transport to use TCPConn.SetLinger(0). Tweaking the network configuration is not enough, we also have to change the client’s TCP connection to set SO_LINGER to 0 when create connections. See this article for detailed explanation.
After all these tweak, the performance of the final tool is really good, I can generate 200 concurrent connection with 400 max QPS using a single 2.4G Hz Xeon core.

Another beautiful feature of Go is that you can develop in your Mac OS and directly cross compile a Linux working binary on your Dev machine and deploy-by-copying-a-single-file. Sweet!

You can checkout the full working source code here: https://github.com/jiaz/simpleloadclient.
