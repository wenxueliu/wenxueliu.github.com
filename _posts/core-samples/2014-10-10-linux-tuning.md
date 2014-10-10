---
layout: post
category : web
tagline: "linux network tuning"
tags : [linux, tunning, network]
---
{% include JB/setup %}

Clone From [Here](http://fasterdata.es.net/host-tuning/linux/)

##Linux Network Tuning

This page contains a quick reference guide for Linux 2.6+ tuning for Data Transfer hosts connected at speeds of 1Gbps or higher. Note that most of the tuning settings described here will actually decrease performance of hosts connected at rates of OC3 (155 Mbps) or less, such as home users on Cable/DSL connections.

For a detailed explanation of some of the advice on this page, see the [Linux Tuning Expert page](fasterdata.es.net/host-tuning/linux/expert/). Note that the settings on this page are not attempting to achieve full 10G with a single flow. These settings assume you are using tools that support parallel streams, or have multiple data transfers occurrin in parallel, and want to have fair sharing between the flows.  As such the maximum values are 2 to 4 times less than what would be required to support a single stream.  As an example, a 10Gbps flow across a 100ms network requires 120MB of buffering.  Most data movement applications, such as GridFTP, would employ 2-8 streams to do this efficiently and to guard against congestive packet loss.  Setting your 10Gbps capable host to consume a maximum of 32M - 64M per socket ensures that parallel streams work well, and do not consume a majority of system resources.

If you are trying to optimize for a single flow, see the [tuning advice for test / measurement hosts](http://fasterdata.es.net/host-tuning/linux/test-measurement-host-tuning/) page.


###General Approach

To check what setting your system is using, use 'sysctl name' (e.g.: 'sysctl net.ipv4.tcp_rmem'). To change a setting use 'sysctl -w'. To make the setting permanent add the setting to the file 'sysctl.conf'.

###TCP tuning

Like most modern OSes, Linux now does a good job of [auto-tuning](http://fasterdata.es.net/host-tuning/background/#t1) the TCP buffers, but the default maximum Linux TCP buffer sizes are still too small. Here are some example sysctl.conf commands for different types of hosts.

For a host with a 10G NIC, optimized for network paths up to 100ms RTT(get by ping), and for friendlyness to single and parallel stream tools, add this to /etc/sysctl.conf

		# allow testing with buffers up to 64MB
		net.core.rmem_max = 67108864
		net.core.wmem_max = 67108864
		# increase Linux autotuning TCP buffer limit to 32MB
		net.ipv4.tcp_rmem = 4096 87380 33554432
		net.ipv4.tcp_wmem = 4096 65536 33554432
		# increase the length of the processor input queue
		net.core.netdev_max_backlog = 30000
		# recommended default congestion control is htcp
		net.ipv4.tcp_congestion_control=htcp
		# recommended for hosts with jumbo frames enabled
		net.ipv4.tcp_mtu_probing=1

For a host with a 10G NIC optimized for network paths up to 200ms RTT, and for friendlyness to single and parallel stream tools, or a 40G NIC up on paths up to 50ms RTT:

		# allow testing with buffers up to 128MB
		net.core.rmem_max = 134217728
		net.core.wmem_max = 134217728
		# increase Linux autotuning TCP buffer limit to 64MB
		net.ipv4.tcp_rmem = 4096 87380 67108864
		net.ipv4.tcp_wmem = 4096 65536 67108864
		# increase the length of the processor input queue
		net.core.netdev_max_backlog = 250000
		# recommended default congestion control is htcp
		net.ipv4.tcp_congestion_control=htcp
		# recommended for hosts with jumbo frames enabled
		net.ipv4.tcp_mtu_probing=1

Notes: you should leave `net.tcp_mem` alone, as the defaults are fine. A number of performance experts say to also increase `net.core.optmem_max` to match `net.core.rmem_max` and `net.core.wmem_max`, but we have not found that makes any difference. Some experts also say to set `net.ipv4.tcp_timestamps` and `net.ipv4.tcp_sack` to 0, as doing that reduces CPU load. We strongly disagree with that recommendation for WAN performance, as we have observed that the default value of 1 helps in more cases than it hurts, and can help a lot.

Linux supports pluggable [congestion control algorithms](http://fasterdata.es.net/host-tuning/background/#t1). To get a list of congestion control algorithms that are available in your kernel (kernal  2.6.20+), run:

		sysctl net.ipv4.tcp_available_congestion_control

If cubic and/or htcp are not listed try the following, as most distributions include them as loadable kernel modules:

		/sbin/modprobe tcp_htcp
		/sbin/modprobe tcp_cubic

NOTE: There seem to be bugs in both bic and cubic for a number of versions of the Linux kernel up to version 2.6.33. We recommend using htcp with older kernels to be safe. To set the congestion control do:

		sysctl -w net.ipv4.tcp_congestion_control=htcp

If you are using [Jumbo Frames](http://fasterdata.es.net/network-tuning/mtu-issues/), we recommend setting tcp_mtu_probing = 1 to help avoid the problem of [MTU black holes](http://en.wikipedia.org/wiki/Path_MTU_Discovery). Setting it to 2 sometimes causes performance problems.

###UDP Tuning

####NIC Tuning

This can be added to /etc/rc.local to be run at boot time:

    # increase txqueuelen for 10G NICS
    /sbin/ifconfig ethN txqueuelen 10000

Note that this might have adverse affects for a 10G host sending to a 1G host or slower.

We also note that we have seen about a 30% performance hit using VLANS with Linux with some hosts/NICS, as it can break the hardware offload capabilities. Myricom mentions this [here](https://www.myricom.com/software/myri10ge/347-what-is-the-performance-impact-of-vlan-tagging-with-the-myri10ge-driver.html).

