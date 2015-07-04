--- 
layout: post
title: "国产某jQuery UI库破解"
wordpress_id: 6
wordpress_url: http://www.yunjing.me/?p=6
date: 2015-07-04 15:02:00 +08:00
category: jquery
tags: 
 - jquery
 - miniui
 - 破解
---
国产有一个基于jQuery开发的UI库，功能很强大，质量也不错。用了一段时间之后，已经沉溺其中，但无奈授权费用太高（据说开发者都要1W多），所以，只能默默打开Chrome的开发者工具。

### 简介
此UI库主要提供一些很方便的UI控件，全部使用javascript开发，最棒的是DataGrid控件，简直是爱不释手。然而，从官网下载的库文件包，所有，代码都经验混淆、压缩、还有隐藏的代码，最让人不爽的是，在每过一段时间，随机性地alert一个模式框。想象一下，当你还在开心地刷副本，突然，因为不小心打开使用这个库的网站，你不得不被切出来看着这个对话框的时候，心里有多窝火。

### 目标
彻底解决掉这个对话框

### 思路
本文基于官方于2015.06.10发布的V3.6版本进行研究。

1、将压缩的代码按标准的缩进展开；  
使用Chrome的开发者工具，查看主js文件，左下角落里有一个 {} 按钮，点击即可。  
2、使用Sublime复制这一段代码并保存为.js文件，语法高亮获得；  
3、将所有以|分隔的数字数组全部展开；  
一般都会用eval调用解密后的代码文本，使用console.log代替eval打印出来；  
4、所有以|分隔的数字数组的变量名，在js文件里到处都有校验，都要移除掉；  
使用这个变量名全文搜索一下，但凡是判断charCodeAt的地方统统删除掉；  
5、意外之外的HOOK：凡是判断某一个字符串里是否包含/r字符的语句统统删掉；  
原本以为进行到第4步就万事大吉，想不到打开页面竟然一片空白，Oh my God，一天多的手动替换全白费了。
继续下断，找啊找，终于，发现一个很神奇的 判断一个函数的toString里indexOf一个/r字符是否!= -1，这有问题，不科学。
全文搜索一个这个代表/r的字符，然后，有好多类似的代码片段。
<!--more-->
<pre class="brush: javascript" line="1">
lOoO1l = oOOloo = oO1oll = olOoo = l1ol11 = oll000 = ollO11 = o11101 = l1o10l = OoOol1 = ol0110 = o011l1 = o0O0l0 = OO0loO = O10l1O = l0OO10 = o0010l = o11oo0 = lll0lO = oo00lo = window;
o0l = ll0 = lOO = oOO = oOo = l10 = Ool = loO1o0 = lOl = OO1 = oO0 = ol000O = O00 = OoO = o01 = "toString";
Oo1 = ooo = o00 = Oo0 = oOl = l0Ol00 = ool = O1lolo = l1l = l00 = OOl = Ol0 = oll = oO0O11 = ol0o10 = "indexOf";
llo = o0o = Oll = l1O = o1O = OlO = lO000o = O10ol1 = OOO0OO = lO0 = "\r";
</pre>
统统删除，这个好多，不能手动删除，要累死人。可以参考以下正则表达式（Sublime适用）：
<!--more-->
<pre class="brush: text" line="1">
^\s+if\s\([10oOl]+\[[10oOl]+\]\(\)\[(Oo1|ooo|o00|Oo0|oOl|l0Ol00|ool|O1lolo|l1l|l00|OOl|Ol0|oll|oO0O11|ol0o10)\]\((llo|o0o|Oll|l1O|o1O|OlO|lO000o|O10ol1|OOO0OO|lO0)\).*\s+return;.*\s
</pre>

删除完毕之后，熟悉的页面终于出现~

免责声明：本文仅供学习研究使用，不得用于任何商业用途，否则后果自负。