---
layout: post
category : lessons
tagline: "Supporting tagline"
tags : [intro, beginner, jekyll, tutorial]
---
{% include JB/setup %}

Linux的top或者ps都可以查看进程的cpu利用率，那为什么还需要了解这个细节呢。这篇文章呢有如下两个原因：

* 希望在脚本中，能够以“非阻塞”的方式获取进程cpu利用率
* ps无法获得进程当前时刻的CPU利用率;top则需要至少1秒才能获得进程当前的利用率

###proc文件系统

/proc文件系统是一个伪文件系统，它只存在内存当中，而不占用外存空间。它以文件系统的方式为内核与进程提供通信的接口。用户和应用程序可以通过/proc 得到系统的信息，并可以改变内核的某些参数。由于系统的信息，如进程，是动态改变的，所以用户或应用程序读取 /proc 目录中的文件时，proc文件系统是动态从系统内核读出所需信息并提交的。

/proc目录中有一些以数字命名的目录，它们是进程目录。系统中当前运行的每一个进程在/proc下都对应一个以进程号为目录名的目录/proc/pid，它们是读取进程信息的接口。此外，在 Linux 2.6.0-test6 以上的版本中 /proc/pid 目录中有一个 task 目录，/proc/pid/task 目录中也有一些以该进程所拥有的线程的线程号命名的目录/proc/pid/task/tid，它们是读取线程信息的接口。


####/proc/cpuinfo

该文件中存放了有关 cpu的相关信息(型号，缓存大小等)。cat /proc/cpuinfo 查看

####/proc/stat

该文件包含了所有 CPU 活动的信息，该文件中的所有值都是从系统启动开始累计到当前时刻，可以“非阻塞”的输出。获得一定时间间隔的两次统计就可以计算出这段时间内的进程CPU利用率。不同内核版本中该文件的格式可能不大一致，以下通过实例来说明数据该文件中各字段的含义。

	$cat /proc/stat
	cpu  922559 82 359766 9960650 67334 5 38924 0 0 0
	cpu0 334945 16 106216 2374733 7639 0 34477 0 0 0
	cpu1 126040 13 68996 2618019 4428 3 3096 0 0 0
	cpu2 339321 34 105543 2356402 30727 0 1094 0 0 0
	cpu3 122251 18 79010 2611495 24540 0 256 0 0 0
    intr 1012557 45 3695 0 0 0 0 0 0 1 1369 0 0 ...
	....
	ctxt 120671034
	btime 1409189302
	processes 16029
	procs_running 2
	procs_blocked 0
	softirq 23167628 0 7193084 7548 373791 290124 0 3840166 5635839 14757 5812319

这里都统一使用 1/USER_HZ 为一个时间片，多数情况下 USER_HZ 都是取值100，所以这里的一个时间片就是10ms。可以通过系统调用sysconf(_SC_CLK_TCK)来获得准确USER_HZ的取值。

第一行的数值表示的是 CPU 总的使用情况，所以我们只要用第一行的数字计算就可以了。第一行各列[数值的含义](http://www.linuxhowtos.org/System/procstat.htm)：

* user (922559)    	从系统启动开始累计到当前时刻，用户态的CPU时间（单位：jiffies） ，不包含 nice值为负进程。1jiffies=0.01秒
* nice (82)      	从系统启动开始累计到当前时刻，nice值为负的进程所占用的CPU时间（单位：jiffies）
* system (359766)  	从系统启动开始累计到当前时刻，核心时间（单位：jiffies）
* idle (9960650)   	从系统启动开始累计到当前时刻，除硬盘IO等待时间以外其它等待时间（单位：jiffies）
* iowait (67334) 	从系统启动开始累计到当前时刻，硬盘IO等待时间（单位：jiffies）
* irq (5)   		从系统启动开始累计到当前时刻，硬中断时间（单位：jiffies）(since 2.6.0-test4)
* softirq (38924)   从系统启动开始累计到当前时刻，软中断时间（单位：jiffies）(since 2.6.0-test4)


* intr  中断的信息，第一个为自系统启动以来，发生的所有的中断的次数；然后每个数对应一个特定的中断自系统启动以来所发生的次数。
* ctxt  自系统启动以来CPU发生的上下文交换的次数。
* btime 从系统启动到现在为止的时间，单位为秒。
* processes (total_forks) 自系统启动以来所创建的任务的个数目。
* procs_running 当前运行队列的任务的数目。
* procs_blocked 当前被阻塞的任务的数目。

在Linux的内核中，有一个全局变量：Jiffies。 Jiffies代表时间。它的单位随硬件平台的不同而不同。系统里定义了一个常数HZ，代表每秒种最小时间间隔的数目。这样jiffies的单位就是 1/HZ。Intel平台jiffies的单位是1/100秒，这就是系统所能分辨的最小时间间隔了。每个CPU时间片，Jiffies都要加1。 CPU的利用率就是用执行用户态+系统态的 Jiffies 除以总的 Jifffies 来表示。

	CPU消耗 = cat /proc/stat|grep "cpu "|awk '{for(i=2;i<=NF;i++)j+=$i;print "cpu_total_slice " j;}'

####/proc/[pid]/stat

该文件包含了某一进程所有的活动的信息，该文件中的所有值都是从系统启动开始累计到当前时刻。以下通过实例数据来说明该文件中各字段的含义。

	$cat /proc/1/stat
	1 (init) S 0 1 1 0 -1 4219136 28147 1146579 27 1969 42 80 20317 2329 20 0 1 0 3 27717632 741 18446744073709551615 1 1 0 0 0 0 0 4096 536962595 18446744073709551615 0 0 17 0 0 0 174 0 0 0 0 0 0 0 0 0 0

输出的第14、15、16、17列分别对应进程用户态 CPU消耗、内核态的消耗、用户态等待子进程的消耗、内核态等待子进程的消耗(man proc)

	CPU消耗 = cat /proc/[pid]/stat | awk '{print "cpu_process_total_slice " $14+$15+$16+$17}'

####/proc/[pid]/task/[tid]/stat

该文件包含了某一进程所有的活动的信息，该文件中的所有值都是从系统启动开始累计到当前时刻。该文件的内容格式以及各字段的含义同/proc/[pid]/stat文件。该文件中的tid字段表示的不再是进程号，而是linux中的轻量级进程(lwp)，即我们通常所说的线程。

proc下有其他文件，更多见 `man 5 proc`


###CPU 利用率计算

从以上分析可以看出，是没有某个时刻CPU利用率的说法的，也就没法获得某个时刻的CPU利用率。这就像物理中的"速度"的概念，
没有某一时刻速度的概念，速度一定是一个时间段之内的。那么要"非阻塞"计算某个进程CPU利用率，则需要取两次事件间隔进行
计算，这两次事件间隔的操作可以是非阻塞的。计算办法如下：

* 时刻A，计算操作系统总CPU时间片消耗total_cpu_slice_A；计算进程总CPU时间片消耗；total_process_slice_A
* 时刻B，计算操作系统总CPU时间片消耗total_cpu_slice_B；计算进程总CPU时间片消耗；total_process_slice_B

B时刻就可以"非阻塞"的计算这段时间进程的CPU利用率了：

	100% \* (total_process_slice_B-total_process_slice_A)/(total_cpu_slice_B-total_cpu_slice_A)

####单核情况下Cpu使用率的计算

**总的Cpu使用率计算**

1 采样两个足够短的时间间隔的Cpu快照，分别记作t1,t2，其中t1、t2的结构均为：

	(user、nice、system、idle、iowait、irq、softirq)的 7 元组;

	CPU时间=user+system+nice+idle+iowait+irq+softirq

2 计算总的Cpu时间片totalCpuTime

* 把第一次的所有cpu使用情况求和，得到j1;
* 把第二次的所有cpu使用情况求和，得到j2;
* s2 - s1 得到这个时间间隔内的所有时间片，即totalCpuTime = j2 - j1 ;

3 计算空闲时间idle

* idle : 对应第四列的数据，用第二次的第四列 - 第一次的第四列即可
* idle : 第二次的第四列 - 第一次的第四列

4 计算cpu使用率

pcpu =100 \* (total-idle)/total

####某一进程Cpu使用率的计算

1 采样 /proc/cpu/stat 文件两个足够短的时间间隔的cpu快照与进程快照，

* 每一个cpu快照均为 (user、nice、system、idle、iowait、irq、softirq、stealstolen、guest)的9元组;
* 每一个进程快照均为 (utime、stime、cutime、cstime)的4元组；

2．计算两个时刻的总的cpu时间与进程的cpu时间，分别记作：totalCpuTime1、totalCpuTime2、processCpuTime1、processCpuTime2
3．计算该进程的cpu使用率pcpu = 100 \* ( processCpuTime2 – processCpuTime1) / (totalCpuTime2 – totalCpuTime1) (按100%计算，如果是多核情况下还需乘以cpu的个数);

**某一线程Cpu使用率的计算**

1 采样

* 采样两个足够短的时间隔的cpu快照与线程快照，
* 每一个cpu快照均为(user、nice、system、idle、iowait、irq、softirq、stealstealon、guest)的9元组;
* 每一个线程快照均为 (utime、stime)的2元组；

2 计算出两个时刻的总的cpu时间与线程的cpu时间，分别记作：totalCpuTime1、totalCpuTime2、threadCpuTime1、threadCpuTime2

3 计算该线程的cpu使用率pcpu = 100 × ( threadCpuTime2 – threadCpuTime1) / (totalCpuTime2 – totalCpuTime1) (按100%计算，如果是多核情况下还需乘以cpu的个数);

####多核情况下cpu使用率的计算

###附录

####Bash 脚本

	#!/bin/sh
	##echo user nice system idle iowait irq softirq
	CPULOG_1=$(cat /proc/stat | grep 'cpu ' | awk '{print $2" "$3" "$4" "$5" "$6" "$7" "$8}')
	SYS_IDLE_1=$(echo $CPULOG_1 | awk '{print $4}')
	Total_1=$(echo $CPULOG_1 | awk '{print $1+$2+$3+$4+$5+$6+$7}')

	sleep 5

	CPULOG_2=$(cat /proc/stat | grep 'cpu ' | awk '{print $2" "$3" "$4" "$5" "$6" "$7" "$8}')
	SYS_IDLE_2=$(echo $CPULOG_2 | awk '{print $4}')
	Total_2=$(echo $CPULOG_2 | awk '{print $1+$2+$3+$4+$5+$6+$7}') 

	SYS_IDLE=`expr $SYS_IDLE_2 - $SYS_IDLE_1`

	Total=`expr $Total_2 - $Total_1`
	SYS_USAGE=`expr $SYS_IDLE/$Total\*100 |bc -l`

	SYS_Rate=`expr 100-$SYS_USAGE |bc -l`

	Disp_SYS_Rate=`expr "scale=3; $SYS_Rate/1" |bc`
	echo $Disp_SYS_Rate%

####Perl 脚本

	#!/usr/bin/perl
	use warnings;

	$SLEEPTIME=5;

	if (-e "/tmp/stat") {
		unlink "/tmp/stat";
	}
	open (JIFF_TMP, "&gt;&gt;/tmp/stat") || die "Can't open /proc/stat file!\n";
	open (JIFF, "/proc/stat") || die "Can't open /proc/stat file!\n";
	@jiff_0=;
	print JIFF_TMP $jiff_0[0] ;
	close (JIFF);

	sleep $SLEEPTIME;

	open (JIFF, "/proc/stat") || die "Can't open /proc/stat file!\n";  @jiff_1=;
	print JIFF_TMP $jiff_1[0];
	close (JIFF);
	close (JIFF_TMP);

	@USER=`awk '{print \$2}' "/tmp/stat"`;
	@NICE=`awk '{print \$3}' "/tmp/stat"`;
	@SYSTEM=`awk '{print \$4}' "/tmp/stat"`;
	@IDLE=`awk '{print \$5}' "/tmp/stat"`;
	@IOWAIT=`awk '{print \$6}' "/tmp/stat"`;
	@IRQ=`awk '{print \$7}' "/tmp/stat"`;
	@SOFTIRQ=`awk '{print \$8}' "/tmp/stat"`;

	$JIFF_0=$USER[0]+$NICE[0]+$SYSTEM[0]+$IDLE[0]+$IOWAIT[0]+$IRQ[0]+$SOFTIRQ[0];
	$JIFF_1=$USER[1]+$NICE[1]+$SYSTEM[1]+$IDLE[1]+$IOWAIT[1]+$IRQ[1]+$SOFTIRQ[1];
	$SYS_IDLE=($IDLE[0]-$IDLE[1]) / ($JIFF_0-$JIFF_1) \* 100;  $SYS_USAGE=100 - $SYS_IDLE;

	printf ("The CPU usage is %1.2f%%\n",$SYS_USAGE);

####ps 命令

通过ps命令可以查看系统中相关进程的Cpu使用率的信息。以下在linux man文档中对ps命令输出中有关cpu使用率的解释：

	CPU usage is currently expressed as the percentage of time spent running during the entire lifetime of a process. This is not ideal, and it does not conform to the standards that ps otherwise conforms to. CPU usage is unlikely to add up to exactly 100%.

	%cpu   cpu utilization of the process in "##.#" format. It is the CPU time used                           divided by the time the process has been running (cputime/realtime ratio),                           expressed as a percentage. It will not add up to 100% unless you are lucky.


结论：ps命令算出来的cpu使用率相对于进程启动时的平均值，随着进程运行时间的增大，该值会趋向于平缓。


####top命令

通过top命令可以查看系统中相关进程的实时信息（cpu使用率等）。以下是man文档中对top命令输出中有关进程cpu使用率的解释。

* P  --  Last used CPU (SMP)

A number representing the last used processor. In a true  SMP  environment  this  will  likely change  frequently  since  the  kernel intentionally uses weak affinity.  Also, the very act of running top may break this weak affinity and cause more processes to  change  CPUs  more  often (because of the extra demand for cpu time).

* %CPU  --  CPU usage

The task’s share of the elapsed CPU time since the last screen update, expressed as a percent-age of total CPU time.  In a true SMP environment, if  Irix mode is Off, top will operate in Solaris mode where a task’s cpu usage will be divided by the total number of CPUs.


###参考

http://www.orczhou.com/index.php/2013/10/how-linux-caculate-cpu-usage-for-process/ 

http://www.penglixun.com/tech/system/how_to_calc_load_cpu.html
