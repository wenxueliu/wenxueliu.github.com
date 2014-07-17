---
layout: post
category : linux
tagline: "帮助"
tags : [ linux ]
---
{% include JB/setup %}

linux 获取帮助的三条途径

###man page

* h  获得man程序的帮助
* j 向下一行   
* k 向上一行
* f 向下翻页
* b 向上翻页
* g   手册第一页
* G  手册最后一页
* /string  查找字符   n 向下查找  N向上查找

注：同一个命令可能有不同的手册页。比如 man kill  和man 2 kill返回不同内容，更多man man

###info 

info 比 man 更强大，命令都存储在/usr/share/info

###软件包 README 或 /usr/share/  或 /usr/local/share
