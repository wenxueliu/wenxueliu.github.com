---
layout: post
category : linux
tagline: "linux 工具汇总"
tags : [linux, tools]
---
{% include JB/setup %}

cool tools

* firefox   addon vimperator is so powerful
* tmux   :  远程开发,管理必备利器
* fish   : 开箱即用的 shell
* ngrep  : 捕获 http 包，方便调试 Http 相关网络问题
* ansible : 自动化部署软件
* jenkins : 自动化构建
* iftop   : 查看网络流量
* ranger  : 命令行下的文件管理系统
* docker    linux container more lighter than virutal host
* vagrant   manage you virtual host by virtualbox
* watch
* mplayer
* cloc
* backup-manager
* autojump



* jq        json with shell
* mdp
* gunplot
* xmind     notebook graph
* smark     makrdown GUI editor
* wiznote   notebook
* gitbook   make markdown to pdf book
* pdsh
* rsync

查询

* env
* alias  unalias
* tty
* who
* which
* locate
* whoami
* whereis
* date
* time
* curl ip.cn  ident.me ipinfo.io ifconfig.me
* last reboot : 查看启动时间
* who -b

用户管理

* chmod
* useradd
* usedel
* adduser
* chown
* last
* lastb

文件相关

* ls
* cat
* less
* more
* head
* tail
* cp
* mv
* mkdir
* rmdir
* rm
* touch
* > file

* lsof

lsof -lnPR +c0 -Di +f fgn 我最喜欢的lsof参数前缀。
查看被活动进程删除的文件: lsof -lnPR +c0 -Di +f fgn +L1 -X
针对crash dump直接使用: lsof -lnPR +c0 -Di +f fgn +L1 -X -k unix.0 -m vmcore.0
查看所有套接字: lsof -lnPR +c0 -Di +f fgn -oo1 -i

* fuse
* file
* paste
* od
* rsync
* sync 缓冲区内容写入磁盘
* df
* dd
* tr
* flock


* md5sum

系统配置

* sysctl
* nice
* renice
* ulimit
* service
* chkconfig
* chrt
* host
* mcelog
* taskset : 设置进程和 CPU 的亲和性
* cset: cset shield -c 1-3 ; cset shield -k on
* cpuid : CPU 详细信息
* update-rc.d : 开机启动服务管理
* top  press f press j
* lshw 查看硬件配置, cpu, memory, iface


配置管理

puppet
chef
ansible

双机热备

MHA
keeplived

负载均衡

LVS
Nginx

文本分析工具

* sort
* uniq
* wc
* sed
* awk
* xargs
* tee
* find
* grep
* ack 替代 grep
* cut
* paste
* od
* du
* expect
* split
* pv
* exiftool //view and edit image
* stdbuf

代码分析

callgrap
cflow
tree2dox
dot
gprof
valgrind
gprof2dot

远程登陆与执行

* ssh
* pssh
* fabric
* pgm
* Fabric
* scp
* vnc
* rdesktop


编辑

* vim
* autojump

演示

* impress.js
* Hovercraft
* Strut
* markdown
* pandoc


网络

* brctl
* iftop(need install)
* nethogs
* iperf
* pv
* nc
* bwm-ng
* tcpflow
* ngrep :ngrep -d any -W byline 'a_search_string' 'dst port 80

* ip
* ifconfig
* ethtool
* iwconfig
* iwlist
* ping
* route
* telnet
* arp
* traceroute
* iptraf
* dig
* host
* nslookup
* tc        :linux traffic control
* netperf
* qperf     : RHEL 网络带宽基准测试

* wireshark
* air-ng
* nmap     : sudo nmap -PT -A -p 80,22 -e wlan0 -sS -sV 192.168.0.1
* tcpdump
* iptables
* hping3

* curl
* httpie
* wget
* http-prompt : pip install http-prompt
* aria2 : 下载工具

* wrk
* ab
* Jemeter
* Ngrinder
* httpload
* webbench
* siege
* autobench
* httperf
* sysbench


性能

* dstat
* vmstat
* mpstat  //apt-get install sysstat
* iostat
* pidstat 监控进程, 查看各个线程的 CPU 消耗
* htop
* top
* /proc/stat
* /proc/meminfo
* /proc/slabinfo
* /proc//maps
* ss
* ps
* netstat
* netstat –tlnp
* pstree
* cpulimit : 限制 CPU 的使用率
* cgroup : 限制进程的使用比率

磁盘

* swapon -s
* swapoff
* dumpe2fs  查看磁盘分区
* fdisk
* df
* fsck
* mkfs
* mount -a -l
* umount
* chroot
* tune2fs
* blktrace
* blkid
* smartmontools apt-get install smartmontools
* fio
* blktrace

版本管理

* git

编译

* ar 静态库
* ranlib  创建静态库的重要工具
* ldd   动态库生成

调试

* nm
* pmap
* objdump 分析目标内容  反汇编
* ldd
* pldd
* pstack
* ipcs
* ipcrm

诊断

* systemtap
* ktap
* perf
* strace
* dmesg
* ftrace
* trace-cmd
* ltrace
* opensnoop : opensnoop -p $PID 程序实时打开文件. 比 strace 快. (需要 4.4+ 内核)


压缩及解压

tar  打包(不同与压缩) 
tar tvf file.tar  查看打包的文件 
tar xvf file.tar  解包
tar rvf file.tar file2  将文件file2加如file.tar 
tar -zcvf file.tar.gz  myfile/ 将myfile下的文件全部压缩 
tar -ztvf file.tar.gz  查看压缩文件 
tar -jxvf file.tar.bz2

gzip bzip2 zip unzip

包管理

* apt
* dpkg
* aptitude
* rpm
* yum
* pacman

进程

* kill
* xkill
* jobs
* bg
* fg
* nohup
* &
* pgrep
* pkill
* fuser
* trap

日志

kafka
sentry : 日志聚合, 报警

论文管理

Mendeley Desktop
Reaya paper

参考文献

jabref

书籍管理

calibre

其他

crontab -l
Byobu


直播

webRTC
HLS(http live streaming)
RTMP
p2p
vokoscreen : 屏幕录制

单机性能测试

sysbench    CPU
fio         磁盘IO
mbw         内存带宽
netperf     网卡带宽


附录


    mplayer -demuxer rawvideo -rawvideo w=176:h=144:format=yuy2 qvga.yuv -loop 0
    mplayer -fps 30 test.264
    mplayer -rawvideo format=help

http://man.linuxde.net/
http://linux.51yip.com/
http://blog.51yip.com/manual/shell/index.html

在 linux上获取随机数据的标准做法是从/dev/(u)random中读取。
当entropy pool被耗尽时，读取/dev/random将产生阻塞，直至
收集到新的随机数据，这减慢了随机数产生的速率。读取
/dev/urandom不会产生阻塞，它会重用internal pool以产生更多
伪随机数。更多时候你需要的是/dev/urandom，而不是/dev/random。
