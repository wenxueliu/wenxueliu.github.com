
The performance of your network can have a significant impact on the general performance and reliability of the rest of your environment. If different applications and services are waiting for data over the network, or your clients are having trouble connecting or receiving the information, then you need to address these issues.

Performance issues can also affect the reliability of your applications and environment, and can both be triggered by network faults, and in some cases they can even be the reason for a network fault. To understand and diagnose network issues, you first need to unde the nature of the issue; usually the problem will be related either to a latency or a bandwidth issue.

In general, network performance issues are often tied to the underlying hardware; you cannot exceed the physical limits of the network environment.

This article looks at the following steps involved in identifying performance issues:

* Getting a baseline performance level
* Determining where the problem lies
* Getting statistics
* Identifying the bottleneck

Understanding network metrics

To understand and diagnose performance issues, you first need to determine your baseline performance level. Let's first introduce two of the key concepts used in determining baseline performance: network latency and network bandwidth.


####Network latency

The network latency is the time between sending a request to a destination and the destination actually receiving the sent packet. As a metric for network performance, increased latency is a good indicator of a busy network, as it either indicates that the number of packets being transmitted exceeds the capacity, or that the senders of data are having to wait before either transmission or re-transmission.

Network latency can also be introduced when the complexity of the network and the number of hosts or gateways that a packet has to travel through increases. The length of cable between points can also have an effect on the latency. For long distances, traditional copper cable will always be slower than using a fibre optic connection.

Network latency is also different from application latency. Network latency deals exclusively with the transmission of packets over the network, while application latency refers to the delay between the application receiving a request and its ability to respond.

####Network bandwidth

Bandwidth is a measure of the number of packets that can be transmitted over a network during a specific period of time. The bandwidth affects how much data can be transmitted, and will either limit the transmission of data to one host to the practical maximum supported by the network connection, or will limit the aggregate transmission rate when dealing with multiple simultaneous connections.

The network bandwidth should, in theory, never change, unless you change the networking interface and hardware. The major variable within network bandwidth is in the number of hosts using the network at any given time.

For example, a 1GB Ethernet interface can talk 1GB to one other network host, 100MB to ten simultaneous hosts, or 10MB to 100 hosts. In reality, of course, the sustained bandwidth is not often required. There will be many hundreds of smaller requests from a number of different hosts over a period of time, and so the available bandwidth of a server can appear much greater than the sum of the client bandwidth.

####Getting network statistics

Before you can identify whether there is a problem within your network, you first need to have a baseline performance on which to base your assumptions. To do this you must check the various parameters -- latency, performance and any tests relevant to your network application environment -- to determine the performance and then monitor and compare this over time.

When performing the baseline networking tests, you should do them under controlled conditions. Ideally, you should perform them under both isolated (meaning with no other network traffic) and with typical network traffic to give you the two baselines:

    For the isolated monitoring, you should check the performance between the server and one or more clients when there is no other traffic on the network. This means either shutting down other services, or, ideally, putting the server and client into an isolated network environment completely separate (but identical to) your standard network environment
    For the standard monitoring, you should have the clients and servers attached to your standard network, and have the normal background traffic working, but all application-specific traffic (such as e-mail, file serving, Web serving) disabled, except on the server that you are testing.

For the actual testing process, there are a number of standard tools and tests that you can perform to determine your baseline values. 

####Measuring latency

The ping sends an echo packet to the device, and expects the device to echo the packet contents back. During the process, ping can monitor the time it takes to send and receive the response, which can be an effective method of measuring the response time of the echo process. In the simplest form, you can send an echo request to a host and find out the response time:

		$ ping example

		PING example.example.pri (192.168.0.2): 56 data bytes
		64 bytes from 192.168.0.2: icmp_seq=0 ttl=64 time=0.169 ms
		64 bytes from 192.168.0.2: icmp_seq=1 ttl=64 time=0.167 ms
		^C
		--- example.example.pri ping statistics ---
		2 packets transmitted, 2 packets received, 0% packet loss
		round-trip min/avg/max/stddev = 0.167/0.168/0.169/0.001 ms

You need to use Control-C to stop the ping process. On Solaris and AIX®, you need to use the -s option to send more than one echo packet and get the timing information. For getting baseline figures, you can use the -c option (on Linux®) to specify the count. On Solaris/AIX, you must specify the packet size (the default is 56 bytes), and the number of packets to send so that you do not have to manually terminate the process. You can then use this to extract the timing information automatically:

		$ ping -s example 56 10
		PING example: 56 data bytes
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=0. time=0.143 ms
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=1. time=0.163 ms
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=2. time=0.146 ms
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=3. time=0.134 ms
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=4. time=0.151 ms
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=5. time=0.107 ms
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=6. time=0.142 ms
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=7. time=0.136 ms
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=8. time=0.143 ms
		64 bytes from example.example.pri (192.168.0.2): icmp_seq=9. time=0.103 ms

		----example PING Statistics----
		10 packets transmitted, 10 packets received, 0% packet loss
		round-trip (ms)  min/avg/max/stddev = 0.103/0.137/0.163/0.019

The example above was made during a quiet period on the network. If the host being checked (or the network itself) was busy during the testing period, the ping times could be increased significantly. However, ping alone is not necessarily an indicator of a problem, but it can occasionally give you a quick idea if there is something that needs to be identified.

It is possible to switch off support for ping, and so you should ensure that you can reach the host before using it as a verification that a host is available.

Ideally, you should track the ping times between specific hosts over a period of time, and even continually, so that you can track the average response times and then identify where to start looking.


####Diagnosing a problem

Typically, you will identify a network problem only when a network-related application fails for some reason. However, it is important to identify that the problem is network related and not a problem elsewhere.

First, you should try to reach the machine using ping. If the machine does not respond to a ping request, and other network communication does not work, then your first option should be to check the physical cables and make sure everything is still connected.

If you can still connect to the machine, but the ping time is increased, then you need to determine where the problem lies. An increase in ping times can in rare cases be related to the load on the machine, but more often than not indicates an issue with the network.

Once you get a long ping time from one machine, you should run ping from another machine on the network, ideally on a different network switch, to find out if the problem is related to the specific machine or the network.

Checking network stats

If the ping times are higher than you expect, then you should start to get some basic statistics about the network interface you are using to see if the problem is related to the network interface, or a specific protocol.

Under Linux, you can get some basic network statistic information by using the ifconfig tool:

		$ ifconfig eth1
		eth1      Link encap:Ethernet  HWaddr 00:1a:ee:01:01:c0  
				  inet addr:192.168.0.2  Bcast:192.168.3.255  Mask:255.255.252.0
				  inet6 addr: fe80::21a:eeff:fe01:1c0/64 Scope:Link
				  UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
				  RX packets:7916836 errors:0 dropped:78489 overruns:0 frame:0
				  TX packets:6285476 errors:0 dropped:0 overruns:0 carrier:0
				  collisions:0 txqueuelen:1000 
				  RX bytes:11675092739 (10.8 GiB)  TX bytes:581702020 (554.7 MiB)
				  Interrupt:16 Base address:0x2000 
				  
The important rows are those beginning RX and TX, which show information about the packets sent and received. The packets value is a simple count of the packets transferred. The errors, dropped, and overruns figures show how many of the packets indicated some kind of fault. A high number of dropped packets in comparison to the packets sent probably indicate that the network is busy.

You can also get extended statistic information on all platforms by using the netstat tool. Under Linux, the tool provides more specific base protocol statistics, such as the packet transmissions for TCP-IP and UDP packet types. Again, the information contains some basic statistics.

		$ netstat -s
		Ip:
			8437387 total packets received
			1 with invalid addresses
			0 forwarded
			0 incoming packets discarded
			8437383 incoming packets delivered
			6820934 requests sent out
			6 reassemblies required
			3 packets reassembled ok
		... ... ... 

Under Solaris and other UNIX variants, the information provided by netstat differs depending upon the platform. For example, under Solaris, you get detailed statistics for each protocol, and separate information for IPv4 and IPv6 connections (see Listing 9). The output in the listing has been truncated.

$ netstat -s

RAWIP   rawipInDatagrams    =   440     rawipInErrors       =     0
        rawipInCksumErrs    =     0     rawipOutDatagrams   =    91
        rawipOutErrors      =     0

UDP     udpInDatagrams      = 15756     udpInErrors         =     0
        udpOutDatagrams     = 16515     udpOutErrors        =     0

TCP     tcpRtoAlgorithm     =     4     tcpRtoMin           =   400
        tcpRtoMax           = 60000     tcpMaxConn          =    -1
... ... ... 
...

In all cases, you are looking for a high level of error packets, retransmissions, or dropped packet transmission, all of which indicate that the network is busy. If the error rate is excessively high compared to the packets transmitted or received, then it may indicate a fault with the network hardware.



















###Tunning Network

####Why Tunning

The default value of rmem_max and wmem_max is about 128 KB in most Linux distributions, which may be enough for a low-latency general purpose network environment or for apps such as DNS / Web server. However, if the latency is large, the default size might be too small. Please note that the following settings going to increase memory usage on your server.









Only when the queue of pending connections is full, AND another SYN packet (connection attempt) is received, AND it has been more than a minute since the last warning message, does the kernel send the warning message you have seen ("sending cookies"). SYN cookies are sent even when the warning message isn't; the warning message is just to give you a heads up that the issue hasn't gone away.

Put another way, if you turn off SYN cookies, the message will go away. That is only going to work out for you if you are no longer being SYN flooded.

####TCP Receive Window (RWIN)

In computer networking, RWIN (TCP Receive Window) is the maximum amount of data that a computer will accept before acknowledging the sender. In practical terms, that means when you download say a 20 MB file, the remote server does not just send you the 20 MB continuously after you request it. When your computer sends the request for the file, your computer tells the remote server what your RWIN value is; the remote server then starts streaming data at you until it reaches your RWIN value, and then the server waits until your computer acknowledges that you received that data OK. Once your computer sends the acknowledgement, then the server continues to send more data in chunks of your RWIN value, each time waiting for your acknowledgment before proceeding to send more.

Now the crux of the problem here is with what is called latency, or the amount of time that it takes to send and receive packets from the remote server. Note that latency will depend not only on how fast the connection is between you and the remote server, but it also includes all additional delays, such as the time that it takes for the server to process your request and respond. You can easily find out the latency between you and the remote server with the ping command. When you use ping, the time that ping reports is the round-trip time (RTT), or latency, between you and the remote server.

When I ping google.com, I typically get a latency of 100 msec. Now if there were no concept of RWIN, and thus my computer had to acknowledge every single packet sent between me and google, then transfer speed between me and them would be simply the (packet size)/RTT. Thus for a maximum sized packet (my MTU as we learned above), my transfer speed would be:

		1492 bytes/.1 sec = 14,920 B/sec or 14.57 KiB/sec

That is pathetically slow considering that my connection is 3 Mb/sec, which is the same as 366 KiB/sec; so I would be using only about 4% of my available bandwidth. Therefore, we use the concept of RWIN so that a remote server can stream data to me without having to acknowledge every single packet and slow everything down to a crawl.

Note that the TCP receive window (RWIN) is independent of the MTU setting. RWIN is determined by the BDP (Bandwidth Delay Product) for your internet connection, and BDP can be calculated as:

		BDP = max bandwidth of your internet connection (Bytes/second) * RTT (seconds)

Therefore RWIN does not depend on the TCP packet size, and TCP packet size is of course limited by the MTU (Maximum Transmission Unit).

Before we change RWIN, use the following command to get the kernel variables related to RWIN:

		sysctl -a 2> /dev/null | grep -iE "_mem |_rmem|_wmem"

Note the space after the _mem is deliberate, don't remove it or add other spaces elsewhere between the quotes.

You should get the following three variables:

		net.ipv4.tcp_rmem = 4096 87380 2584576
		net.ipv4.tcp_wmem = 4096 16384 2584576
		net.ipv4.tcp_mem = 258576 258576 258576

The variable numbers are in bytes, and they represent the minimum, default, and maximum values for each of those variables.

		net.ipv4.tcp_rmem = Receive window memory vector
		net.ipv4.tcp_wmem = Send window memory vector
		net.ipv4.tcp_mem = TCP stack memory vector

Note that there is no exact equivalent variable in Linux that corresponds to RWIN, the closest is the net.ipv4.tcp_rmem variable. The variables above control the actual memory usage (not just the TCP window size) and include memory used by the socket data structures as well as memory wasted by short packets in large buffers. The maximum values have to be larger than the BDP (Bandwidth Delay Product) of the path by some suitable overhead.

To try and optimize RWIN, first use ping to send the maximum size packet your connection allows (MTU) to some distant server. Since my MTU is 1492, the ping command payload would be 1492-28=1464. Thus:


		$ ping -s 1464 -c5 google.com

		PING google.com (64.233.167.99) 1464(1492) bytes of data.
		64 bytes from py-in-f99.google.com (64.233.167.99): icmp_seq=1 ttl=237 (truncated)
		64 bytes from py-in-f99.google.com (64.233.167.99): icmp_seq=2 ttl=237 (truncated)
		64 bytes from py-in-f99.google.com (64.233.167.99): icmp_seq=3 ttl=237 (truncated)
		64 bytes from py-in-f99.google.com (64.233.167.99): icmp_seq=4 ttl=237 (truncated)
		64 bytes from py-in-f99.google.com (64.233.167.99): icmp_seq=5 ttl=237 (truncated)

		--- google.com ping statistics ---
		5 packets transmitted, 5 received, 0% packet loss, time 3999ms
		rtt min/avg/max/mdev = 101.411/102.699/105.723/1.637 ms

Note though that you should run the above test several times at different times during the day, and also try pinging other destinations. You'll see RTT might vary quite a bit.

But for the above example, the RTT average is about 103 msec. Now since the maximum speed of my internet connection is 3 Mbits/sec, then the BDP is:


		(3,000,000 bits/sec) * (.103 sec) * (1 byte/8 bits) = 38,625 bytes

Thus I should set the default value in net.ipv4.tcp_rmem to about 39,000. For my internet connection, I've seen RTT as bad as 500 msec, which would lead to a BDP of 187,000 bytes.

Therefore, I could set the max value in net.ipv4.tcp_rmem to about 187,000. The values in net.ipv4.tcp_wmem should be the same as net.ipv4.tcp_rmem since both sending and receiving use the same internet connection. And since net.ipv4.tcp_mem is the maximum total memory buffer for TCP transactions, it is usually set to the the max value used in net.ipv4.tcp_rmem and net.ipv4.tcp_wmem.

And lastly, there are two more kernel TCP variables related to RWIN that you should set:

		sysctl -a 2> /dev/null | grep -iE "rcvbuf|save"

which returns:

		net.ipv4.tcp_no_metrics_save = 1
		net.ipv4.tcp_moderate_rcvbuf = 1

Note enabling net.ipv4.tcp_no_metrics_save (setting it to 1) means have Linux optimize the TCP receive window dynamically between the values in net.ipv4.tcp_rmem and net.ipv4.tcp_wmem. And enabling net.ipv4.tcp_moderate_rcvbuf removes an odd behavior in the 2.6 kernels, whereby the kernel stores the slow start threshold for a client between TCP sessions. This can cause undesired results, as a single period of congestion can affect many subsequent connections.

Before you change any of the above variables, try going to http://www.speedtest.net or a similar website and check the speed of your connection. Then temporarily change the variables by using the following command with your own computed values:

		sudo sysctl -w net.ipv4.tcp_rmem="4096 39000 187000" net.ipv4.tcp_wmem="4096 39000 187000" net.ipv4.tcp_mem="187000 187000 187000" net.ipv4.tcp_no_metrics_save=1 net.ipv4.tcp_moderate_rcvbuf=1

Once you tweak the values to your liking, you can make them permanent by adding them to /etc/sysctl.conf as follows:

		net.ipv4.tcp_rmem=4096 39000 187000
		net.ipv4.tcp_wmem=4096 39000 187000
		net.ipv4.tcp_mem=187000 187000 187000
		net.ipv4.tcp_no_metrics_save=1
		net.ipv4.tcp_moderate_rcvbuf=1

And then do the following command to make the changes permanent:

   		sudo sysctl -p



####BDP

TCP depends on several factors for performance. Two of the most important are the link bandwidth (the rate at which packets can be transmitted on the network) and the round-trip time, or RTT (the delay between a segment being sent and its acknowledgment from the peer). These two values determine what is called the Bandwidth Delay Product (BDP).

Given the link bandwidth rate and the RTT, you can calculate the BDP, but what does this do for you? It turns out that the BDP gives you an easy way to calculate the theoretical optimal TCP socket buffer sizes (which hold both the queued data awaiting transmission and queued data awaiting receipt by the application). If the buffer is too small, the TCP window cannot fully open, and this limits performance. If it's too large, precious memory resources can be wasted. If you set the buffer just right, you can fully utilize the available bandwidth. Let's look at an example:

	BDP = link_bandwidth * RTT
	
	Note:
		RTT: The ping program will give you the round trip time (RTT) for the network link, which is twice the delay, 

If your application communicates over a 100Mbps local area network with a 50 ms RTT, the BDP is:

	100MBps * 0.050 sec / 8 = 0.625MB = 625KB
	
Note: I divide by 8 to convert from bits to bytes communicated.

So, set your TCP window to the BDP, or 625KB. But the default window for TCP on Linux 2.6 is 110KB, which limits your bandwidth for the connection to 2.2MBps, as I've calculated here:

	throughput = window_size / RTT 	110KB / 0.050 = 2.2MBps

If instead you use the window size calculated above, you get a whopping 12.5MBps, as shown here:

	625KB / 0.050 = 12.5MBps
	
That's quite a difference and will provide greater throughput for your socket. So you now know how to calculate the optimal socket buffer size for your socket. But how do you make this change?


###Setting the TCP Buffer Size

There are two TCP settings to consider: the default TCP buffer size and the maximum TCP buffer size. A user-level program can modify the default buffer size, but the maximum buffer size requires administrator privileges. Note that most of today's Unix-based OSes by default have a maximum TCP buffer size of only 256K. Windows does not have a maximum buffer size by default, but the administrator may set one. It is necessary to change both the send and receive TCP buffers. Changing only the send buffer will have no effect, as TCP negotiates the buffer size to be the smaller of the two. This means that it is not necessary to set both the send and receive buffer to the optimal value. A common technique is to set the buffer in the server quite large (for example, 1,024K) and then let the client determine and set the correct "optimal" value for that network path. To set the TCP buffer,


we want to enable RFC 1323 Window Scaling and increase the TCP window size to 1 MB. To do this, we'll add the following lines to /etc/sysctl.conf and issue sudo sysctl -p to apply the changes immediately.

    net.core.rmem_max = 16777216
    net.core.wmem_max = 16777216
    net.core.rmem_default = 1048576
    net.core.wmem_default = 1048576
    net.ipv4.tcp_rmem = 4096 1048576 16777216
    net.ipv4.tcp_wmem = 4096 1048576 16777216
    net.ipv4.tcp_congestion_control = bic
    net.ipv4.tcp_window_scaling = 1
    net.ipv4.tcp_timestamps = 1

	Note: you should leave tcp_mem alone. The defaults are fine.
	
Another thing you can do to help increase TCP throughput with 1GB NICs is to increase the size of the interface queue. For paths with more than 50 ms RTT, a value of 5000-10000 is recommended. To increase txqueuelen, do the following:

	[root@server1 ~] ifconfig eth0 txqueuelen 5000
	
As before, we're setting the maximum buffer size large and the default window size to 1 MB. RFC 1323 is enabled via net.ipv4.tcp_window_scaling and net.ipv4.tcp_timestamps. These options are probably on by default, but it never hurts to force them via /etc/sysctl.conf. Finally, we are choosing BIC as our TCP Congestion Control Algorithm. Again, that value is most likely the default on your system (especially any kernel version after 2.6.12).





* /proc/sys/net/core/rmem_default 	"110592" 	
	Defines the default receive window size; for a large BDP, the size should be larger.

* /proc/sys/net/core/rmem_max 	"110592" 	
	Defines the maximum receive window size; for a large BDP, the size should be larger.

* /proc/sys/net/core/wmem_default 	"110592" 	
	Defines the default send window size; for a large BDP, the size should be larger.

* /proc/sys/net/core/wmem_max 	"110592" 	
	Defines the maximum send window size; for a large BDP, the size should be larger.

* /proc/sys/net/ipv4/tcp_window_scaling 	"1" 	
	Enables window scaling as defined by RFC 1323; must be enabled to support windows larger than 64KB.

* /proc/sys/net/ipv4/tcp_sack 	"1" 	
	Enables selective acknowledgment, which improves performance by selectively acknowledging packets received out of order (causing the sender to retransmit only the missing segments); should be enabled (for wide area network communication), but it can increase CPU utilization.

* /proc/sys/net/ipv4/tcp_fack 	"1" 	
	Enables Forward Acknowledgment, which operates with Selective Acknowledgment (SACK) to reduce congestion; should be enabled.

* /proc/sys/net/ipv4/tcp_timestamps 	"1" 	
	Enables calculation of RTT in a more accurate way (see RFC 1323) than the retransmission timeout; should be enabled for performance.

* /proc/sys/net/ipv4/tcp_mem 	"24576 32768 49152" 	
	Determines how the TCP stack should behave for memory usage; each count is in memory pages (typically 4KB). The first value is the low threshold for memory usage. The second value is the threshold for a memory pressure mode to begin to apply pressure to buffer usage. The third value is the maximum threshold. At this level, packets can be dropped to reduce memory usage. Increase the count for large BDP (but remember, it's memory pages, not bytes).

* /proc/sys/net/ipv4/tcp_wmem 	"4096 16384 131072" 	
	Defines per-socket memory usage for auto-tuning. The first value is the minimum number of bytes allocated for the socket's send buffer. The second value is the default (overridden by wmem_default) to which the buffer can grow under non-heavy system loads. The third value is the maximum send buffer space (overridden by wmem_max).

* /proc/sys/net/ipv4/tcp_rmem 	"4096 87380 174760" 	
	Same as tcp_wmem except that it refers to receive buffers for auto-tuning.

* /proc/sys/net/ipv4/tcp_low_latency 	"0" 	
	Allows the TCP/IP stack to give deference to low latency over higher throughput; should be disabled.

* /proc/sys/net/ipv4/tcp_westwood 	"0" 	
	Enables a sender-side congestion control algorithm that maintains estimates of throughput and tries to optimize the overall utilization of bandwidth; should be enabled for WAN communication. This option is also useful for wireless interfaces, as packet loss may not be caused by congestion.

* /proc/sys/net/ipv4/tcp_bic 	"1" 	
Enables Binary Increase Congestion for fast long-distance networks; permits better utilization of links operating at gigabit speeds; should be enabled for WAN communication.

* /proc/sys/net/ipv4/tcp_congestion_control  bic


As with any tuning effort, the best approach is experimental in nature. Your application behavior, processor speed, and availability of memory all affect how these parameters will alter performance. In some cases, what you think should be beneficial can be detrimental (and vice versa). So, try an option and then check the result. In other words, trust but verify.

Bonus tip: A word about persistent configuration. Note that if you reboot a GNU/Linux system, any tunable kernel parameters that you changed revert to their default. To make yours the default parameter, use the file /etc/sysctl.conf to configure the parameters at boot-time for your configuration.



####More

Other kernel settings that help with the overall server performance when it comes to network traffic are the following:

TCP_FIN_TIMEOUT - This setting determines the time that must elapse before TCP/IP can release a closed connection and reuse its resources. During this TIME_WAIT state, reopening the connection to the client costs less than establishing a new connection. By reducing the value of this entry, TCP/IP can release closed connections faster, making more resources available for new connections. 

	[root@server:~]# echo 30 > /proc/sys/net/ipv4/tcp_fin_timeout
Addjust this in the presense of many connections sitting in the TIME_WAIT state

	[root@server:~]# echo 30 > /proc/sys/net/ipv4/tcp_keepalive_intvl

TCP_KEEPALIVE_INTERVAL - This determines the wait time between isAlive interval probes. 

	[root@server:~]# echo 5 > /proc/sys/net/ipv4/tcp_keepalive_probes
	
TCP_KEEPALIVE_PROBES - This determines the number of probes before timing out. 

	[root@server:~]# echo 1 > /proc/sys/net/ipv4/tcp_tw_recycle

TCP_TW_REUSE - This allows reusing sockets in TIME_WAIT state for new connections when it is safe from protocol viewpoint. Default value is 0 (disabled). It is generally a safer alternative to tcp_tw_recycle 

	[root@server:~]# echo 1 > /proc/sys/net/ipv4/tcp_tw_reuse

Note: The tcp_tw_reuse setting is particularly useful in environments where numerous short connections are open and left in TIME_WAIT state, such as web servers and loadbalancers. Reusing the sockets can be very effective in reducing server load. 

Starting in Linux 2.6.7 (and back-ported to 2.4.27), linux includes alternative congestion control algorithms beside the traditional 'reno' algorithm. These are designed to recover quickly from packet loss on high-speed WANs.

There are a couple additional sysctl settings for kernels 2.6 and newer:

Not to cache ssthresh from previous connection:
	
	net.ipv4.tcp_no_metrics_save = 1

To increase this for 10G NICS:

	net.core.netdev_max_backlog = 30000

Starting with version 2.6.13, Linux supports pluggable congestion control algorithms . The congestion control algorithm used is set using the sysctl variable net.ipv4.tcp_congestion_control, which is set to bic/cubic or reno by default, depending on which version of the 2.6 kernel you are using.

To get a list of congestion control algorithms that are available in your kernel (if you are running 2.6.20 or higher), run:

	[root@server1 ~] # sysctl net.ipv4.tcp_available_congestion_control

The choice of congestion control options is selected when you build the kernel. The following are some of the options are available in the 2.6.23 kernel:

* reno: Traditional TCP used by almost all other OSes. (default)
* cubic: CUBIC-TCP (NOTE: There is a cubic bug in the Linux 2.6.18 kernel used by Redhat Enterprise Linux 5.3 and Scientific Linux 5.3. Use 2.6.18.2 or higher!)
* bic: BIC-TCP
* htcp: Hamilton TCP
* vegas: TCP Vegas
* westwood: optimized for lossy networks

If cubic and/or htcp are not listed when you do 'sysctl net.ipv4.tcp_available_congestion_control', try the following, as most distributions include them as loadable kernel modules:


	[root@server1 ~] # /sbin/modprobe tcp_htcp 
	[root@server1 ~] # /sbin/modprobe tcp_cubic

For long fast paths, I highly recommend using cubic or htcp. Cubic is the default for a number of Linux distributions, but if is not the default on your system, you can do the following:

 	[root@server1 ~] # sysctl -w net.ipv4.tcp_congestion_control=cubic

On systems supporting RPMS, You can also try using the ktune RPM, which sets many of these as well.


If you have a load server that has many connections in TIME_WAIT state decrease the TIME_WAIT interval that determines the time that must elapse before TCP/IP can release a closed connection and reuse its resources. This interval between closure and release is known as the TIME_WAIT state or twice the maximum segment lifetime (2MSL) state. During this time, reopening the connection to the client and server cost less than establishing a new connection. By reducing the value of this entry, TCP/IP can release closed connections faster, providing more resources for new connections. Adjust this parameter if the running application requires rapid release, the creation of new connections, and a low throughput due to many connections sitting in the TIME_WAIT state:

	[root@host1 ~]# echo 5 > /proc/sys/net/ipv4/tcp_fin_timeout
	
If you are often dealing with SYN floods the following tunning can be helpful:



	[root@host1 ~]# sysctl -w net.ipv4.tcp_max_syn_backlog="16384"
	
the maximum number of remembered connection requests, which still have not received an acknowledgment from connecting clients.

	[root@host1 ~]# sysctl -w net.ipv4.tcp_synack_retries="1"
	
the number of SYN+ACK packets sent before the kernel gives up on the connection. To open the other side of the connection, the kernel sends a SYN with a piggybacked ACK on it, to acknowledge the earlier received SYN. This is part 2 of the three-way handshake.


	[root@host1 ~]# sysctl -w net.ipv4.tcp_max_orphans="400000"
	
the maximum number of TCP sockets not attached to any user file handle, held by system. If this number is exceeded orphaned connections are reset immediately and warning is printed. This limit exists only to prevent simple DoS attacks, you _must_ not rely on this or lower the limit artificially, but rather increase it (probably, after increasing installed memory), if network conditions require more than default value, and tune network services to linger and kill such states more aggressively.

More information on tuning parameters and defaults for Linux 2.6 are available in the file ip-sysctl.txt, which is part of the 2.6 source distribution.

Warning on Large MTUs: If you have configured your Linux host to use 9K MTUs, but the connection is using 1500 byte packets, then you actually need 9/1.5 = 6 times more buffer space in order to fill the pipe. In fact some device drivers only allocate memory in power of two sizes, so you may even need 16/1.5 = 11 times more buffer space!

And finally a warning for both 2.4 and 2.6: for very large BDP paths where the TCP window is > 20 MB, you are likely to hit the Linux SACK implementation problem. If Linux has too many packets in flight when it gets a SACK event, it takes too long to located the SACKed packet, and you get a TCP timeout and CWND goes back to 1 packet. Restricting the TCP buffer size to about 12 MB seems to avoid this problem, but clearly limits your total throughput. Another solution is to disable SACK.

Starting with Linux 2.4, Linux implemented a sender-side autotuning mechanism, so that setting the optimal buffer size on the sender is not needed. This assumes you have set large buffers on the receive side, as the sending buffer will not grow beyond the size of the receive buffer.

However, Linux 2.4 has some other strange behavior that one needs to be aware of. For example: The value for ssthresh for a given path is cached in the routing table. This means that if a connection has a retransmission and reduces its window, then all connections to that host for the next 10 minutes will use a reduced window size, and not even try to increase its window. The only way to disable this behavior is to do the following before all new connections (you must be root):

1 	[root@server1 ~] # sysctl -w net.ipv4.route.flush=1

Lastly I would like to point out how important it is to have a sufficient number of available file descriptors, since pretty much everything on Linux is a file.

To check your current max and availability run the following:

	[root@host1 ~]# sysctl fs.file-nr
	fs.file-nr = 197600 0 3624009

The first value (197600) is the number of allocated file handles.
The second value (0) is the number of unused but allocated file handles. And the third value (3624009) is the system-wide maximum number of file handles. It can be increased by tuning the following kernel parameter:

	[root@host1 ~]# echo 10000000 > /proc/sys/fs/file-max
	
To see how many file descriptors are being used by a process you can use one of the following:

	[root@host1 ~]# lsof -a -p 28290
	[root@host1 ~]# ls -l /proc/28290/fd | wc -l

The 28290 number is the process id. 


###other parameter 

* Open files

Since we deal with a lot of file handles (each TCP socket requires a file handle), we need to keep our open file limit high. The current value can be seen using ulimit -a (look for open files). We set this value to 999999 and hope that we never need a million or more files open. In practice we never do.

We set this limit by putting a file into /etc/security/limits.d/ that contains the following two lines:

*	soft	nofile	999999
*	hard	nofile	999999

(side node: it took me 10 minutes trying to convince Markdown that those asterisks were to be printed as asterisks)

If you don’t do this, you’ll run out of open file handles and could see one or more parts of your stack die.

* Ephemeral Ports

The second thing to do is to increase the number of Ephemeral Ports available to your application. By default this is all ports from 32768 to 61000. We change this to all ports from 18000 to 65535. Ports below 18000 are reserved for current and future use of the application itself. This may change in the future, but is sufficient for what we need right now, largely because of what we do next.

* TIME_WAIT state

TCP connections go through various states during their lifetime. There’s the handshake that goes through multiple states, then the ESTABLISHED state, and then a whole bunch of states for either end to terminate the connection, and finally a TIME_WAIT state that lasts a really long time. If you’re interested in all the states, read through the netstat man page, but right now the only one we care about is the TIME_WAIT state, and we care about it mainly because it’s so long.

By default, a connection is supposed to stay in the TIME_WAIT state for twice the msl. Its purpose is to make sure any lost packets that arrive after a connection is closed do not confuse the TCP subsystem (the full details of this are beyond the scope of this article, but ask me if you’d like details). The default msl is 60 seconds, which puts the default TIME_WAIT timeout value at 2 minutes. Which means you’ll run out of available ports if you receive more than about 400 requests a second, or if we look back to how nginx does proxies, this actually translates to 200 requests per second. Not good for scaling.

We fixed this by setting the timeout value to 1 second.

I’ll let that sink in a bit. Essentially we reduced the timeout value by 99.16%. This is a huge reduction, and not to be taken lightly. Any documentation you read will recommend against it, but here’s why we did it.

Again, remember the point of the TIME_WAIT state is to avoid confusing the transport layer. The transport layer will get confused if it receives an out of order packet on a currently established socket, and send a reset packet in response. The key here is the term established socket. A socket is a tuple of 4 terms. The source and destination IPs and ports. Now for our purposes, our server IP is constant, so that leaves 3 variables.

Our port numbers are recycled, and we have 47535 of them. That leaves the other end of the connection.

In order for a collision to take place, we’d have to get a new connection from an existing client, AND that client would have to use the same port number that it used for the earlier connection, AND our server would have to assign the same port number to this connection as it did before. Given that we use persistent HTTP connections between clients and nginx, the probability of this happening is so low that we can ignore it. 1 second is a long enough TIME_WAIT timeout.

The two TCP tuning parameters were set using sysctl by putting a file into /etc/sysctl.d/ with the following:

		net.ipv4.ip_local_port_range = 18000    65535
		net.ipv4.netfilter.ip_conntrack_tcp_timeout_time_wait = 1


* Connection Tracking

The next parameter we looked at was Connection Tracking. This is a side effect of using iptables. Since iptables needs to allow two-way communication between established HTTP and ssh connections, it needs to keep track of which connections are established, and it puts these into a connection tracking table. This table grows. And grows. And grows.

You can see the current size of this table using sysctl net.netfilter.nf_conntrack_count and its limit using sysctl net.nf_conntrack_max. If count crosses max, your linux system will stop accepting new TCP connections and you’ll never know about this. The only indication that this has happened is a single line hidden somewhere in /var/log/syslog saying that you’re out of connection tracking entries. One line, once, when it first happens.

A better indication is if count is always very close to max. You might think, “Hey, we’ve set max exactly right.”, but you’d be wrong.

What you need to do (or at least that’s what you first think) is to increase max.

Keep in mind though, that the larger this value, the more RAM the kernel will use to keep track of these entries. RAM that could be used by your application.

We started down this path, increasing net.nf_conntrack_max, but soon we were just pushing it up every day. Connections that were getting in there were never getting out.

* nf_conntrack_tcp_timeout_established

It turns out that there’s another timeout value you need to be concerned with. The established connection timeout. Technically this should only apply to connections that are in the ESTABLISHED state, and a connection should get out of this state when a FIN packet goes through in either direction. This doesn’t appear to happen and I’m not entirely sure why.

So how long do connections stay in this table then? The default value for nf_conntrack_tcp_timeout_established is 432000 seconds. I’ll wait for you to do the long division…

Fun times.

I changed the timeout value to 10 minutes (600 seconds) and in a few days time I noticed conntrack_count go down steadily until it sat at a very manageable level of a few thousand.

We did this by adding another line to the sysctl file:

		net.netfilter.nf_conntrack_tcp_timeout_established=600

* Speed bump

At this point we were in a pretty good state. Our beacon collectors ran for months (not counting scheduled reboots) without a problem, until a couple of days ago, when one of them just stopped responding to any kind of network requests. No ping responses, no ACK packets to a SYN, nothing. All established ssh and HTTP connections terminated and the box was doing nothing. I still had console access, and couldn’t tell what was wrong. The system was using less than 1% CPU and less than 10% of RAM. All processes that were supposed to be running were running, but nothing was coming in or going out.

I looked through syslog, and found one obscure message repeated several times.

IPv4: dst cache overflow

Well, there were other messages, but this was the one that mattered.

I did a bit of searching online, and found something about an rt_cache leak in 2.6.18. We’re on 3.5.2, so it shouldn’t have been a problem, but I investigated anyway.

The details of the post above related to 2.6, and 3.5 was different, with no ip_dst_cache entry in /proc/slabinfo so I started searching for its equivalent on 3.5 when I came across Vincent Bernat's post on the IPv4 route cache. This is an excellent resource to understand the route cache on linux, and that’s where I found out about the lnstat command. This is something that needs to be added to any monitoring and stats gathering scripts that you run. Further reading suggests that the dst cache gc routines are complicated, and a bug anywhere could result in a leak, one which could take several weeks to become apparent.

From what I can tell, there doesn’t appear to be an rt_cache leak. The number of cache entries increases and decreases with traffic, but I’ll keep monitoring it to see if that changes over time.


* Window size after idle

Related to the above is the sysctl setting net.ipv4.tcp_slow_start_after_idle. This tells the system whether it should start at the default window size only for new TCP connections or also for existing TCP connections that have been idle for too long (on 3.5, too long is 1 second, but see net.sctp.rto_initial for its current value on your system). If you’re using persistent HTTP connections, you’re likely to end up in this state, so set net.ipv4.tcp_slow_start_after_idle=0 (just put it into the sysctl config file mentioned above).

* Endgame

After changing all these settings, a single quad core vm (though using only one core) with 1Gig of RAM has been able to handle all the load that’s been thrown at it. We never run out of open file handles, never run out of ports, never run out of connection tracking entries and never run out of RAM.

We have several weeks before another one of our beacon collectors runs into the dst cache issue, and I’ll be ready with the numbers when that happens.

Thanks for reading, and let us know how these settings work out for you if you try them out. If you’d like to measure the real user impact of your changes, have a look at our Real User Measurement tool at LogNormal.



参考链接

http://www.softpanorama.org/Commercial_linuxes/Performance_tuning/tcp_performance_tuning.shtml#Some_kernel_limits_recommended_by_Oracle_
http://blog.dubbelboer.com/2012/04/09/syn-cookies.html
http://serverfault.com/questions/294209/possible-syn-flooding-in-log-despite-low-number-of-syn-recv-connections

http://tools.ietf.org/pdf/rfc4987.pdf
http://fasterdata.es.net/host-tuning/linux/


###附录

####高负载网络参数 tunning

    net.ipv4.tcp_syncookies = 1
    
表示开启SYN Cookies。当出现SYN等待队列溢出时，启用cookies来处理，可防范少量SYN攻击，默认为0，表示关闭； 

	net.ipv4.tcp_mem

Don’t touch tcp_mem for two reasons: Firstly, unlike tcp_rmem and tcp_wmem it’s in pages, not bytes, so it’s likely to confuse the hell out of you. Secondly, it’s already auto-tuned very well by Linux based on the amount of RAM.

	net.ipv4.tcp_max_syn_backlog = 4096

Increase the number of outstanding syn requests allowed. Note: some people (including myself) have used tcp_syncookies to handle the problem of too many legitimate outstanding SYNs. I quote the Linux documentation:

  	 	Note, that syncookies is fallback facility. It MUST NOT be used to help highly 
  	 	loaded servers to stand against legal connection rate. If you see synflood 
  	 	warnings in your logs, but investigation shows that they occur because of 
  	 	overload with legal connections, you should tune another parameters until 
  	 	this warning disappear.


	vm.min_free_kbytes = 65536

This tells the kernel to try and keep 64MB of RAM free at all times. It’s useful in two main cases:

* Swap-less machines, where you don’t want incoming network traffic to overwhelm the kernel and force an OOM before it has time to flush any buffers.

* x86 machines, for the same reason: the x86 architecture only allows DMA transfers below approximately 900MB of RAM. So you can end up with the bizarre situation of an OOM error with tons of RAM free.

    vm.swappiness = 0

It’s said that altering swappiness can help you when you’re running under high memory pressure with software that tries to do its own memory management (i.e. MySQL). We’ve had limited success with this and I’d much prefer to use software which doesn’t pretend to know more about your hardware than the OS (i.e. PostgreSQL). Not that I’m bitter.


		# echo 'net.ipv4.tcp_no_metrics_save = 1' >> /etc/sysctl.conf
		
By default, TCP saves various connection metrics in the route cache when the connection closes, so that connections established in the near future can use these to set initial conditions. Usually, this increases overall performance, but may sometimes cause performance degradation. If set, TCP will not cache metrics on closing connections.


		# echo 'net.ipv4.tcp_sack = 1' >> /etc/sysctl.conf
		
 Enable select acknowledgments

tunning 例子

		net.ipv4.tcp_timestamps = 0
		net.ipv4.tcp_sack = 0
		net.ipv4.tcp_rmem = 210000 210000 210000
		net.ipv4.tcp_wmem = 210000 210000 210000
		net.ipv4.tcp_mem = 210000 210000 210000

		net.core.rmem_max = 1073741824
		net.core.wmem_max = 1073741824
		net.core.rmem_default = 1073741824
		net.core.wmem_default = 1073741824
		net.core.optmem_max = 524287
		net.core.netdev_max_backlog = 30000000
		
		
		# tunning tcp stack
		sysctl -w net.ipv4.tcp_fin_timeout=30
		sysctl -w net.ipv4.tcp_keepalive_time=1800
		sysctl -w net.ipv4.tcp_window_scaling=0
		sysctl -w net.ipv4.tcp_sack=0
		sysctl -w net.ipv4.tcp_timestamps=0

		sysctl -w net.ipv4.ip_conntrack_max=524288
		sysctl -w net.ipv4.tcp_syncookies=1

		sysctl -w net.ipv4.icmp_echo_ignore_broadcasts=1


	
		net.ipv4.tcp_syn_retries = 3 # default=5
		net.ipv4.tcp_synack_retries = 3 # default=5
		net.ipv4.tcp_max_syn_backlog = 65536 # default=1024
		net.core.wmem_max = 8388608 # default=124928
		net.core.rmem_max = 8388608 # default=131071
		net.core.somaxconn = 512  # default = 128
		net.core.optmem_max = 81920 # default = 20480
		# buffering
		sysctl -w net.core.wmem_default=229376
		sysctl -w net.core.wmem_max=229376



		sysctl -p 
		net.ipv4.conf.all.rp_filter = 1 
		net.ipv4.ip_forward = 1 
		net.ipv4.conf.default.send_redirects = 1 
		net.ipv4.conf.all.send_redirects = 0 
		net.ipv4.icmp_echo_ignore_broadcasts = 1 
		net.ipv4.conf.default.forwarding = 1 
		net.ipv4.conf.default.proxy_arp = 0 
		kernel.sysrq = 1 
		net.ipv4.conf.eth0.proxy_arp = 1 
		net.core.somaxconn = 4096 
		net.ipv4.tcp_keepalive_intvl = 30 
		net.ipv4.tcp_keepalive_probes = 5 
		net.ipv4.tcp_tw_reuse = 1


		fs.file-max=1048576 
		kernel.shmmax=8589934592 
		net.core.rmem_max = 16777216 
		net.core.wmem_max = 16777216 
		net.core.rmem_default = 1048576 
		net.core.wmem_default = 1048576 
		net.core.netdev_max_backlog=16384 
		net.core.somaxconn=32768 
		net.core.optmem_max = 25165824 
		net.ipv4.tcp_rmem = 4096 1048576 16777216 
		net.ipv4.tcp_wmem = 4096 1048576 16777216 
		net.ipv4.tcp_max_syn_backlog=32768 
		vm.max_map_count=131060


####ab 测试 tips


* ab 工具，如果出现 apr_socket_recv: Connection reset by peer (104)， 可增加 -r 参数

* 默认情况下，ab没有启用gzip压缩功能，所以压力测试的结果会跟实际情况有很大的偏差。要想让ab使用gzip压缩功能，得添加参数 -H 'Accept-Encoding: gzip'

		ab -H 'Accept-Encoding: gzip' www.xxx.com/

* 提示：Benchmarking 127.0.0.1 (be patient)... 使用了与请求的协议不兼容的地址(730047)

如果出现这个问题，那很可能是你使用了apache2.4.1或以上的版本。似乎从2.4.*开始，就使用了ipv6的协议，另一种角度来说，这可能是一个bug，所以检测一下是不是以前把ipv6的相关服务给关了


#####工具

* Dtrace
* tcpjunk
* http://www.opensourcetesting.org/performance.php
* netlog	为应用程序提供一些有关网络性能方面的信息。
* nettimer  为瓶颈链接带宽生成一个度量标准；可以用于协议的自动优化。



####apr_socket_recv: Connection reset by peer (104)


网上有一个解决办法是修改内核参数(被测试服务器)：

		net.ipv4.conf.default.rp_filter = 1

		net.ipv4.conf.all.rp_filter = 1

		net.ipv4.tcp_syncookies = 0

		net.ipv4.tcp_max_syn_backlog = 819200

		net.ipv4.tcp_synack_retries = 1

		net.ipv4.tcp_max_tw_buckets = 819200

	
执行sysctl -p生效


执行以上内核参数修改操作后，ab还是同样的错误，只是被测试的服务器的日志中不会再有以下提示：

		Oct 9 15:07:21 localhost kernel: possible SYN flooding on port 80. Sending cookies.


解决：

1、清除以上设置

2、sysctl -p

3、在测试服务器执行以下操作：

修改源代码后重新编译安装apache

		vim support/ab.c

		1373 return;

		1374 } else {

		1375 //apr_err("apr_socket_recv", status);

		1376 bad++;

		1377 close_connection(c);

		1378 return;

		1379 }


注：只使用修改源码的ab程序测试，不修改内核参数的话，测试服务器日志中会提示Oct 9 15:07:21 localhost kernel: possible SYN flooding on port 80. Sending cookies.但是ab正常运行

