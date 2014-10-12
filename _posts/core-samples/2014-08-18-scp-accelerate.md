---
layout: post
category : tools
tagline: "scp 传输大文件"
tags : [tools, linux]
---
{% include JB/setup %}


摘自[这里](http://www.orczhou.com/index.php/2013/11/make-scp-faster-with-cipher-and-compression/)，还有[这里](http://www.orczhou.com/index.php/2013/11/tranfer-data-faster-on-the-fly/)

当需要在机器之间传输400GB文件的时候，你就会非常在意传输的速度了。默认情况下(约125MB带宽，网络延迟17ms，Intel E5-2430，本文后续讨论默认是指该环境)，scp的速度约为40MB，传输400GB则需要170分钟，约3小时，如果可以加速，则可以大大节约工程师的时间，让攻城师们有更多时间去看个电影，陪陪家人。

###性能环境说明

整个过程有几个阶段：磁盘读取-->打包(tar)-->压缩-->传输-->解压缩-->拆包-->落盘 对应了的速度测试：

####磁盘读取和落盘

	dd if=./sendlog.tar of=/dev/null bs=4096 count=1048576

	dd if=/dev/zero of=./x.zero.file bs=4096 count=1048576

####打包、拆包

	time tar -cf sendlog.tar ./sendlog/

	time tar -xf sendlog.tar

####压缩、解压缩

	压缩工具的比较测试参考：[Gzip vs Bzip2 vs LZMA vs XZ vs LZ4 vs LZO](http://pokecraft.first-world.info/wiki/Quick_Benchmark:_Gzip_vs_Bzip2_vs_LZMA_vs_XZ_vs_LZ4_vs_LZO#Compression_time)

	可以看到，lz4在压缩率上略微逊色(对比pigz)，但是在解压速度上有这惊人的优势。

	time tar -c sendlog/|lz4|pv > /dev/null

	time tar -c sendlog/|lz4|pv|ssh -c arcfour128 -o "MACs umac-64@openssh.com" 10.xxx.xxx.36 "cat - >/dev/null"

	for i in `seq 4 7`; do time tar -c ./sendlog/|lz4 -B$i |pv > /dev/null ;done

	time tar -c sendlog/|pv|lz4 -B4|ssh -c arcfour128 -o"MACs umac-64@openssh.com" 10.xxx.xxx.36 "lz4 -d |tar -xC /u01/backup_supu"

####传输

   	scp

####整体流程

磁盘读取---->打包---->压缩------>传输---->解压缩-->拆包---->落盘
             |->tar   |->gzip    |->ssh   |->gzip  |->tar
                      |->bzip2   |->http  |->bzip
                      |-> ...    |->nc    |->...
                      |->lz4              |->lz4
>400MB/s    >350MB/s  79MB/s     90MB/s   72MB/s    >350MB/s >400MB/s


###可能的影响因子

* 改变ssh加密算法，可以让速度更快；通常，越弱的加密算法，速度越快

* 通常压缩会降低scp速度，但这与数据类型有很大关系，对压缩率非常高的数据启用压缩，可以加速

* 压缩级别对传输效率影响很小

* 用于完整性校验的不同[MAC( message authentication code)](en.wikipedia.org/wiki/Message_authentication_code)算法，对性能约有10%-20%的影响。

所以，简单尝试如下，让你的SCP速度double一下：

	scp -r -c arcfour128 ...
	scp -r -c aes192-cbc ...
	scp -r -c arcfour128 -o "MACs umac-64@openssh.com" ... 

###测试

####加密算法

对比 12 种 ssh 中实现的加密算法传输效率

####压缩

主要是两个因素 压缩算法和压缩比，对比不同的压缩算法和压缩比的传输效率

* 压缩只有在网络传输速度非常慢，以致于压缩后节省的传输时间大于压缩本身的时间，这时才有效果，所以是否启用压缩，需要实际测试
* 压缩比很低的数据，不要再启用压缩(例如已经压缩过的数据、视频等)
* 通常建议，传输前先压缩，而不是使用ssh的压缩；建议使用pigz/lbizp2等并行压缩工具
* 数据中大量重复、空洞，这类适合压缩的数据，可以尝试压缩选项

####混合

对比加密算法和压缩同时使用与前两种的传输效率

	time tar -c sendlog/|lz4|ssh -c arcfour128 -o"MACs umac-64@openssh.com" 10.xxx.xx.36 "lz4 -d |tar -xC /u01/backup_supu"

 	time tar -c sendlog/|pigz -p 16|ssh -c arcfour128 -o"MACs umac-64@openssh.com" 10.xxx.xx.36 "gzip -d|tar -xC /u01/backup_supu"

 对比发现，在压缩方面pigz与lz4并没有太大区别，但是lz4解压速度非常快，所以在这种需要立刻解压的场景下，lz4轻松胜出(bzip2这种就不需要测试了)。

###总结

####关于lz4

lz4是一个让"人见人爱、花见花开"的压缩算法，能够在多核上很好的扩展，压缩速度和压缩比并没有太大优势(pigz)，但是他的解压速度非常惊人，本案例测试lz4的解压是gunzip的3倍(更多的对比测试)。因为压缩时高效的多核利用，再加上惊艳的解压，lz4已经在非常多重要场合使用了：Linux3.11内核实现了LZ4，并可以使用其压缩和解压kernel image HBase:Add an LZ4 compression option to HFile等等(参考)。

对于需要频繁压缩、实时快速解压的场景来说，lz4非常适合。

####结论

	time tar -c sendlog/|pv|lz4 -B4|ssh -c arcfour128 -o"MACs umac-64@openssh.com" 10.xxx.xxx.36 "lz4 -d |tar -xC /u01/backup_supu"



####参考

http://www.neuhalfen.name/2009/02/04/scp_performance_gain_by_using_right_algorithm/

[HPN SSH](http://www.psc.edu/index.php/hpn-ssh)
