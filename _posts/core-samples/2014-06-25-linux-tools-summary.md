---
layout: post
category : linux
tagline: "linux 工具汇总"
tags : [linux, tools]
---
{% include JB/setup %}

cool tools

* tmux
* screen
* ranger
* docker
* vagrant
* mplayer

查询

* env
* alias  unalias
* tty 当期终端
* who
* which
* locate
* whoami
* whereis
* date
* time

用户管理

* chmod
* useradd
* usedel
* adduser
* chown

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

* lsof
* fuse
* file
* paste
* od
* rsync
* sync 缓冲区内容写入磁盘
* df
* dd

系统配置

* sysctl
* nice
* renice
* ulimit

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
* cut
* tee
* paste
* od
* du
* expect

远程登陆与执行

* ssh
* pssh
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

* ip
* ifconfig
* iwconfig
* ping
* route
* telnet
* arp
* traceroute
* iptraf
* dig

* wireshark
* air-ng
* nmap
* tcpdump
* iptables

* curl
* wget

性能

* vmstat
* htop
* top
* /proc/stat
* /proc/meminfo
* /proc/slabinfo
* /proc//maps
* ss
* ps
* netstat
* pstree

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
* dmesg
* strace
* ipcs
* ipcrm


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
