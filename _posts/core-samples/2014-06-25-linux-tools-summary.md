---
layout: post
category : linux
tagline: "linux 工具汇总"
tags : [linux, tools]
---
{% include JB/setup %}

cool tools

* firefox   addon vimperator is so powerful
* vagrant   manage you virtual host by virtualbox
* docker    linux container more lighter than virutal host
* ranger    file manage
* tmux      remote
* screen    remote
* watch
* mplayer
* cloc
* backup-manager


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
* > file

* lsof
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

配置管理

puppet
chef
ansible

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
* tee
* paste
* od
* du
* expect
* split
* pv
* exiftool //view and edit image

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
* bwm-ng

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
* tc

* wireshark
* air-ng
* nmap
* tcpdump
* iptables

* curl
* httpie
* wget

* httpload
* webbench
* ab
* siege
* autobench
* httperf
* sysbench

性能

* vmstat
* mpstat
* iostat
* pidstat
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

附录


    mplayer -demuxer rawvideo -rawvideo w=176:h=144:format=yuy2 qvga.yuv -loop 0
    mplayer -fps 30 test.264
    mplayer -rawvideo format=help

http://man.linuxde.net/
http://linux.51yip.com/
http://blog.51yip.com/manual/shell/index.html
