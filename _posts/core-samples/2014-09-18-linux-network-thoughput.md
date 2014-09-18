---
layout: post
category : tools
tagline: "统计网络吞吐量"
tags : [ linux]
---
{% include JB/setup %}

在Linux中有很多的流量监控工具，它们可以监控、分类网络流量，以花哨的图形用户界面提供实时流量分析报告。大多数这些工具（例如：[ntopng](http://xmodulo.com/2013/10/set-web-based-network-traffic-monitoring-linux.html) ,  [iftop](http://xmodulo.com/2012/06/how-to-install-iftop-on-linux.html) ）都是基于[libpcap](http://www.tcpdump.org/pcap3_man.html) 库的 ，这个函数库是用来截取流经网卡的数据包的，可在用户空间用来监视分析网络流量。尽管这些工具功能齐全，然而基于libpcap库的流量监控工具无法处理高速（Gb以上）的网络接口，原因是由于在用户空间做数据包截取的系统开销过高所致。

在本文中我们介绍一种简单的Shell 脚本，它可以监控网络流量而且不依赖于缓慢的libpcap库。这些脚本支持Gb以上规模的高速网络接口，如果你对“汇聚型”的网络流量感兴趣的话，它们可统计每个网络接口上的流量。

脚本主要是基于[sysfs](http://lwn.net/Articles/31185/)虚拟文件系统，这是由内核用来将设备或驱动相关的信息输出到用户空间的一种机制。网络接口的相关分析数据会通过“/sys/class/net/<ethX>/statistics”输出。

举个例子，eth0的网口上分析报告会输出到这些文件中：

    /sys/class/net/eth0/statistics/rx_packets: 收到的数据包数据
    /sys/class/net/eth0/statistics/tx_packets: 传输的数据包数量
    /sys/class/net/eth0/statistics/rx_bytes: 接收的字节数
    /sys/class/net/eth0/statistics/tx_bytes: 传输的字节数
    /sys/class/net/eth0/statistics/rx_dropped: 收包时丢弃的数据包
    /sys/class/net/eth0/statistics/tx_dropped: 发包时丢弃的数据包

这些数据会根据内核数据发生变更的时候自动刷新。因此，你可以编写一系列的脚本进行分析并计算流量统计。下面就是这样的脚本（感谢 joemiller 提供）。第一个脚本是统计每秒数据量，包含接收（RX）或发送（TX）。而后面的则是一个描述网络传输中的接收（RX）发送(TX)带宽。这些脚本中安装不需要任何的工具。

测量网口每秒数据包：

	#! /usr/bin/env bash
	
	function get_eth_by_sysfs()
	{
		eth=`ls /sys/class/net/`
		echo $eth
	}

	function calculate_thoughput_by_sysfs()
	{
		if [ $1 ]; then
		    gap_secs=$2
		    gap_secs=${gap_secs:=1}
		    while true
		    do
		        recv_begin_bytes=`cat /sys/class/net/$1/statistics/rx_bytes`
		        recv_begin_pack_index=`cat /sys/class/net/$1/statistics/rx_packets`
		        trans_begin_bytes=`cat /sys/class/net/$1/statistics/tx_bytes`
		        trans_begin_pack_index=`cat /sys/class/net/$1/statistics/tx_packets`
		        sleep $gap_secs
		        recv_end_bytes=`cat /sys/class/net/$1/statistics/rx_bytes`
		        recv_end_pack_index=`cat /sys/class/net/$1/statistics/rx_packets`
		        trans_end_bytes=`cat /sys/class/net/$1/statistics/tx_bytes`
		        trans_end_pack_index=`cat /sys/class/net/$1/statistics/tx_packets`

		        recv_thoughput=`expr $recv_end_bytes - $recv_begin_bytes`
		        recv_package_cnt=`expr $recv_end_pack_index - $recv_begin_pack_index`
		        trans_thoughput=`expr $trans_end_bytes - $trans_begin_bytes`
		        trans_package_cnt=`expr $trans_end_pack_index - $trans_begin_pack_index`

		        echo "Recv/Bytes " $recv_thoughput "Packets:" $recv_package_cnt "Trans/Bytes" $trans_thoughput "Packets:" $trans_package_cnt
		    done
		else
		    echo "ethernet-device must be given(eth0,wlan0 etc)"
		fi
	}
	
	get_eth_by_sysfs
	calculate_thoughput_by_sysfs wlan0
	
当然也有从/proc/net 取数据的。具体脚本如下：	

	#! /usr/bin/env bash

	function get_eth_by_proc()
	{
		eths=`cat /proc/net/dev | sed -n "3,$ p" | awk '{print $1}' | sed "s/://"`
		echo $eths
	}

	function calculate_thoughput_by_proc()
	{
		if [ $1 ]; then
		    gap_secs=$2
		    gap_secs=${gap_secs:=1}
		    while true
		    do
		        recv_begin_bytes=`cat /proc/net/dev | grep $1 | awk '{ print $2}'`
		        recv_begin_pack_index=`cat /proc/net/dev | grep $1 | awk '{ print $2}'`
		        trans_begin_bytes=`cat /proc/net/dev | grep $1 | awk '{ print $2}'`
		        trans_begin_pack_index=`cat /proc/net/dev | grep $1 | awk '{ print $2}'`
		        sleep $gap_secs
		        recv_end_bytes=`cat /proc/net/dev | grep $1 | awk '{ print $2}'`
		        recv_end_pack_index=`cat /proc/net/dev | grep $1 | awk '{ print $2}'`
		        trans_end_bytes=`cat /proc/net/dev | grep $1 | awk '{ print $2}'`
		        trans_end_pack_index=`cat /proc/net/dev | grep $1 | awk '{ print $2}'`

		        recv_thoughput=`expr $recv_end_bytes - $recv_begin_bytes`
		        recv_package_cnt=`expr $recv_end_pack_index - $recv_begin_pack_index`
		        trans_thoughput=`expr $trans_end_bytes - $trans_begin_bytes`
		        trans_package_cnt=`expr $trans_end_pack_index - $trans_begin_pack_index`

		        echo "Recv/Bytes " $recv_thoughput "Packets:" $recv_package_cnt "Trans/Bytes" $trans_thoughput "Packets:" $trans_package_cnt
		    done
		else
		    echo "ethernet-device must be given(eth0,wlan0 etc)"
		fi
	}

	get_eth_by_proc
	calculate_thoughput_by_proc wlan0


###参考

[极客范](http://www.geekfan.net/5558/)
