---
layout: post
title: 刁钻而无意义的笔试题
comments: true
categories:
 - 面试题
---

今天在网上看到有人发表了一篇关于一个无意义而刁钻的笔试题，如
```cpp
#include <stdio>

int main( )
{
    http://www.yunjing.me
    printf("http://www.yunjing.me\n");
    return 0;
}
```
让分析这段代码是否存在问题，若有则指出问题所在。

知道猫腻后，就打算用Go语文来装一下，然后，结果很神奇。
```Go
package main

func main() {
    http://www.yunjing.me
    println("http://www.yunjing.me")
}
```

惊叹于Go语言的极简设计理念，这类刁钻的笔试题并无用武之地，Go编译时报错: `label http defined and not used`
