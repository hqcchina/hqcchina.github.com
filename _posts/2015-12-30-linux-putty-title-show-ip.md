--- 
layout: post
title: "Putty登陆后设置标题栏显示IP"
wordpress_id: 8
wordpress_url: http://www.yunjing.me/?p=8
date: 2015-12-30 11:31:00 +08:00
category: tools
tags: 
 - linux
 - putty
---
用putty一个很头痛的问题，就是连接多台服务器后窗口多了，不知道那个窗口对应那吧服务器。所以把IP显示到标题上，就方便了很多了。在网上找到这种方法，不错，不错，呵呵。记录一下，方便以后使用。

把下面的几行脚本追加到 ~/.bashrc(对应 root 用户，也就是 /root/.bashrc 文件)自动脚本的最后。

<!--more-->
<pre class="brush: text" line="1">
# Auto add env parameter $PROMPT_COMMAND when use non-Linux tty login by ssh.
if [ "$SSH_CONNECTION" != '' -a "$TERM" != 'linux' ]; then
declare -a HOSTIP
HOSTIP=`echo $SSH_CONNECTION |awk '{print $3}'`
export PROMPT_COMMAND='echo -ne "\033]0;${USER}@$HOSTIP:[${HOSTNAME%%.*}]:${PWD/#$HOME/~} \007"'
fi
</pre>

重新登陆该服务器(还是用之前的那个用户登陆)，你会发现左上角又可以看到服务器的 IP了。
