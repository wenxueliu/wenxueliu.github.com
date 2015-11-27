---
layout: post
category : linux
tagline: "常用系统设置"
tags : [ setting,linux ]
---
{% include JB/setup %}


###系统最大打开文件描述符数：/proc/sys/fs/file-max

####查看

	$ cat /proc/sys/fs/file-max
	99960

####设置

* 临时性

	# echo 1000000 > /proc/sys/fs/file-max

* 永久性：

	在/etc/sysctl.conf中设置

	fs.file-max = 1000000

###进程最大打开文件描述符数：user limit中nofile的soft limit

####查看

通过ulimit -Sn 设置最大打开文件描述符数的 soft limit，注意soft limit不能大于hard limit（ulimit -Hn可查看hard limit），
另外ulimit -n默认查看的是soft limit，但是

	$ ulimit -n
	1024

	$ ulimit -Hn
	4096

	设置soft limit，必须小于hard limit：

	$ ulimit -Sn
	1024

####设置

* 临时性

	ulimit -Hn 18000  设置 hard limit
	ulimit -Sn 10000  设置 soft limit
	ulimit -n 1800000 同时设置 soft limit 和 hard limit。对于非root用户只能设置比原来小的 hard limit。

	上面的方法只是临时性的，注销重新登录就失效了

* 永久性

	/etc/security/limits.conf中进行设置（需要root权限），可添加如下两行：

	dev soft   nofile          18000
	dev hard   nofile          20000

	表示用户 dev 最大打开文件描述符数的soft limit为18000，hard limit为20000。需要注销之后重新登录才能生效

设置 nofile 的 hard limit 还有一点要注意的就是 hard limit 不能大于/proc/sys/fs/nr_open，假如 hard limit 大于 nr_open，
注销后无法正常登录。可以修改 nr_open 的值：

	# echo 2000000 > /proc/sys/fs/nr_open


注: centos 6 版本, 是先读 /etc/security/limits.conf, 如果 /etc/security/limits.d/ 目录下还有配置文件的话,
会遍历读取里面文件, 所以 /etc/security/limits.d/ 里面的文件里面的配置会覆盖/etc/security/limits.conf的配置.

###查看当前系统使用的打开文件描述符数

	# cat /proc/sys/fs/file-nr
	5664        0        186405

其中第一个数表示当前系统已分配使用的打开文件描述符数，第二个数为分配后已释放的（目前已不再使用），第三个数等于file-max。


###总结：

* 所有进程打开的文件描述符数不能超过/proc/sys/fs/file-max
* 单个进程打开的文件描述符数不能超过user limit中nofile的soft limit
* nofile 的 soft limit 不能超过其hard limit
* nofile 的 hard limit 不能超过/proc/sys/fs/nr_open

###参考

CentOS 下参考 man ulimit 搜索 ulimit 会看到相关文档

