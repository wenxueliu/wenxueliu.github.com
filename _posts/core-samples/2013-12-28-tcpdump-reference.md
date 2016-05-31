---
layout: post
category : linux
comments : false
tags : [ network, tools, tutorial ]
---
{% include JB/setup %}

整理by文刀旋子,本文档既可作为man tcpdump的补充，又可为tcp入门选择性学习。

#tcpdump

说实在的，对于 tcpdump 这个软件来说，你甚至可以说这个软件其实就是个骇客软件， 因为他不但可以分析封包的流向，连封包的内容也可以进行监听， 如果你使用的传输资料是明码的话，不得了，在 router 上面就可能被人家监听走了！ 很可怕呐！所以，我们也要来了解一下这个软件啊！(注：这个 tcpdump 必须使用 root 的身份执行) 


##命令使用

###tcpdump采用命令行方式，它的命令格式为：

    tcpdump [ -AdDeflLnNOpqRStuUvxX ] [ -c count ]
               [ -C file_size ] [ -F file ]
               [ -i interface ] [ -m module ] [ -M secret ]
               [ -r file ] [ -s snaplen ] [ -T type ] [ -w file ]
               [ -W filecount ]
               [ -E spi@ipaddr algo:secret,...  ]
               [ -y datalinktype ] [ -Z user ]
               [ expression ]

##tcpdump的选项介绍

 **NOTE** : 如果是初学关注经常用到的选项 -D -S -f -i -vv -n -q -t -e -w -F -l -r -s -XX

`-A`

    以ASCII码方式显示每一个数据包(不会显示数据包中链路层头部信息). 在抓取包含网页数据的数据包时, 可方便查看数据(nt: 即Handy for capturing web pages).

`-c count`

    tcpdump将在接受到count个数据包后退出.

`-C  file-size` (nt: 此选项用于配合-w file 选项使用)

    该选项使得tcpdump 在把原始数据包直接保存到文件中之前, 检查此文件大小是否超过file-size. 如果超过了, 将关闭此文件,另创一个文件继续用于原始数据包的记录. 新创建的文件名与-w 选项指定的文件名一致, 但文件名后多了一个数字.该数字会从1开始随着新创建文件的增多而增加. file-size的单位是百万字节(nt: 这里指1,000,000个字节,并非1,048,576个字节, 后者是以1024字节为1k, 1024k字节为1M计算所得, 即1M=1024 ＊ 1024 ＝ 1,048,576)

`-d `

    以容易阅读的形式,在标准输出上打印出编排过的包匹配码, 随后tcpdump停止.(nt | rt: human readable, 容易阅读的,通常是指以ascii码来打印一些信息. compiled, 编排过的. packet-matching code, 包匹配码,含义未知, 需补充)

`-dd` 以C语言的形式打印出包匹配码.

`-ddd` 以十进制数的形式打印出包匹配码(会在包匹配码之前有一个附加的'count'前缀).

`-D`

    打印系统中所有tcpdump可以在其上进行抓包的网络接口. 每一个接口会打印出数字编号, 相应的接口名字, 以及可能的一个网络接口描述. 其中网络接口名字和数字编号可以用在tcpdump 的-i flag 选项(nt: 把名字或数字代替flag), 来指定要在其上抓包的网络接口.

    此选项在不支持接口列表命令的系统上很有用(nt: 比如, Windows 系统, 或缺乏 ifconfig -a 的UNIX系统); 接口的数字编号在windows 2000 或其后的系统中很有用, 因为这些系统上的接口名字比较复杂, 而不易使用.

    如果tcpdump编译时所依赖的libpcap库太老,-D 选项不会被支持, 因为其中缺乏 pcap_findalldevs()函数.

`-e`  每行的打印输出中将包括数据包的数据链路层头部信息

`-E  spi@ipaddr algo:secret,...`

    可通过spi@ipaddr algo:secret 来解密IPsec ESP包(nt | rt:IPsec Encapsulating Security Payload,IPsec 封装安全负载, IPsec可理解为, 一整套对ip数据包的加密协议, ESP 为整个IP 数据包或其中上层协议部分被加密后的数据,前者的工作模式称为隧道模式; 后者的工作模式称为传输模式 . 工作原理, 另需补充).

    需要注意的是, 在终端启动tcpdump 时, 可以为IPv4 ESP packets 设置密钥(secret）.

    可用于加密的算法包括des-cbc, 3des-cbc, blowfish-cbc, rc3-cbc, cast128-cbc, 或者没有(none).默认的是des-cbc(nt: des, Data Encryption Standard, 数据加密标准, 加密算法未知, 另需补充).secret 为用于ESP 的密钥, 使用ASCII 字符串方式表达. 如果以 0x 开头, 该密钥将以16进制方式读入.

    该选项中ESP 的定义遵循RFC2406, 而不是 RFC1827. 并且, 此选项只是用来调试的, 不推荐以真实密钥(secret)来使用该选项, 因为这样不安全: 在命令行中输入的secret 可以被其他人通过ps 等命令查看到.

    除了以上的语法格式(nt: 指spi@ipaddr algo:secret), 还可以在后面添加一个语法输入文件名字供tcpdump 使用(nt：即把spi@ipaddr algo:secret,... 中...换成一个语法文件名). 此文件在接受到第一个ESP　包时会打开此文件, 所以最好此时把赋予tcpdump 的一些特权取消(nt: 可理解为, 这样防范之后, 当该文件为恶意编写时,不至于造成过大损害).

`-f`

    显示外部的IPv4 地址时(nt: foreign IPv4 addresses, 可理解为, 非本机ip地址), 采用数字方式而不是名字.(此选项是用来对付Sun公司的NIS服务器的缺陷(nt: NIS, 网络信息服务, tcpdump 显示外部地址的名字时会用到她提供的名称服务): 此NIS服务器在查询非本地地址名字时,常常会陷入无尽的查询循环).

    由于对外部(foreign)IPv4地址的测试需要用到本地网络接口(nt: tcpdump 抓包时用到的接口)及其IPv4 地址和网络掩码. 如果此地址或网络掩码不可用, 或者此接口根本就没有设置相应网络地址和网络掩码(nt: linux 下的 'any' 网络接口就不需要设置地址和掩码, 不过此'any'接口可以收到系统中所有接口的数据包), 该选项不能正常工作.

`-F file`

    使用file 文件作为过滤条件表达式的输入, 此时命令行上的输入将被忽略.

`-i  interface`

    指定tcpdump 需要监听的接口.  如果没有指定, tcpdump 会从系统接口列表中搜寻编号最小的已配置好的接口(不包括 loopback 接口).一但找到第一个符合条件的接口, 搜寻马上结束.

    在采用2.2版本或之后版本内核的Linux 操作系统上, 'any' 这个虚拟网络接口可被用来接收所有网络接口上的数据包(nt: 这会包括目的是该网络接口的, 也包括目的不是该网络接口的). 需要注意的是如果真实网络接口不能工作在'混杂'模式(promiscuous)下,则无法在'any'这个虚拟的网络接口上抓取其数据包.

    如果 -D 标志被指定, tcpdump会打印系统中的接口编号，而该编号就可用于此处的interface 参数.

`-l`

    对标准输出进行行缓冲(nt: 使标准输出设备遇到一个换行符就马上把这行的内容打印出来).在需要同时观察抓包打印以及保存抓包记录的时候很有用. 比如, 可通过以下命令组合来达到此目的:

    ``tcpdump  -l  |  tee dat'' 或者 ``tcpdump  -l   > dat  &  tail  -f  dat''.(nt: 前者使用tee来把tcpdump 的输出同时放到文件dat和标准输出中, 而后者通过重定向操作'>', 把tcpdump的输出放到dat 文件中, 同时通过tail把dat文件中的内容放到标准输出中)


`-L`

    列出指定网络接口所支持的数据链路层的类型后退出.(nt: 指定接口通过-i 来指定)

`-m  module`

    通过module 指定的file 装载SMI MIB 模块(nt: SMI，Structure of Management Information, 管理信息结构MIB, Management Information Base, 管理信息库. 可理解为, 这两者用于SNMP(Simple Network Management Protoco)协议数据包的抓取. 具体SNMP 的工作原理未知, 另需补充).

    此选项可多次使用, 从而为tcpdump 装载不同的MIB 模块.

`-M  secret`  如果TCP 数据包(TCP segments)有TCP-MD5选项(在RFC 2385有相关描述), 则为其摘要的验证指定一个公共的密钥secret.

`-n` 不对地址(比如, 主机地址, 端口号)进行数字表示到名字表示的转换.

`-N` 不打印出host 的域名部分. 比如, 如果设置了此选现, tcpdump 将会打印'nic' 而不是 'nic.ddn.mil'.

`-O` 不启用进行包匹配时所用的优化代码. 当怀疑某些bug是由优化代码引起的, 此选项将很有用.

`-p`

    一般情况下, 把网络接口设置为非'混杂'模式. 但必须注意 , 在特殊情况下此网络接口还是会以'混杂'模式来工作； 从而, '-p' 的设与不设, 不能当做以下选现的代名词:'ether host {local-hw-add}' 或  'ether broadcast'(nt: 前者表示只匹配以太网地址为host 的包, 后者表示匹配以太网地址为广播地址的数据包).

`-q` 快速(也许用'安静'更好?)打印输出. 即打印很少的协议相关信息, 从而输出行都比较简短.

`-R`

    设定tcpdump 对 ESP/AH 数据包的解析按照 RFC1825而不是RFC1829(nt: AH, 认证头, ESP， 安全负载封装, 这两者会用在IP包的安全传输机制中). 如果此选项被设置, tcpdump 将不会打印出'禁止中继'域(nt: relay prevention field). 另外,由于ESP/AH规范中没有规定ESP/AH数据包必须拥有协议版本号域,所以tcpdump不能从收到的ESP/AH数据包中推导出协议版本号.

`-r  file`

    从文件file 中读取包数据. 如果file 字段为 '-' 符号, 则tcpdump 会从标准输入中读取包数据.

`-S`

    打印TCP 数据包的顺序号时, 使用绝对的顺序号, 而不是相对的顺序号.(nt: 相对顺序号可理解为, 相对第一个TCP 包顺序号的差距,比如, 接受方收到第一个数据包的绝对顺序号为232323, 对于后来接收到的第2个,第3个数据包, tcpdump会打印其序列号为1, 2分别表示与第一个数据包的差距为1 和 2. 而如果此时-S 选项被设置, 对于后来接收到的第2个, 第3个数据包会打印出其绝对顺序号:232324, 232325).

`-s  snaplen`

    设置tcpdump的数据包抓取长度为snaplen, 如果不设置默认将会是68字节(而支持网络接口分接头(nt: NIT, 上文已有描述,可搜索'网络接口分接头'关键字找到那里)的SunOS系列操作系统中默认的也是最小值是96).68字节对于IP, ICMP(nt: Internet Control Message Protocol,因特网控制报文协议), TCP 以及 UDP 协议的报文已足够, 但对于名称服务(nt: 可理解为dns, nis等服务), NFS服务相关的数据包会产生包截短. 如果产生包截短这种情况, tcpdump的相应打印输出行中会出现''[|proto]''的标志（proto 实际会显示为被截短的数据包的相关协议层次). 需要注意的是, 采用长的抓取长度(nt: snaplen比较大), 会增加包的处理时间, 并且会减少tcpdump 可缓存的数据包的数量， 从而会导致数据包的丢失. 所以, 在能抓取我们想要的包的前提下, 抓取长度越小越好.把snaplen 设置为0 意味着让tcpdump自动选择合适的长度来抓取数据包.

`-T  type`

    强制tcpdump按type指定的协议所描述的包结构来分析收到的数据包.  目前已知的type 可取的协议为:
    aodv (Ad-hoc On-demand Distance Vector protocol, 按需距离向量路由协议, 在Ad hoc(点对点模式)网络中使用),
    cnfp (Cisco  NetFlow  protocol),  rpc(Remote Procedure Call), rtp (Real-Time Applications protocol),
    rtcp (Real-Time Applications con-trol protocol), snmp (Simple Network Management Protocol),
    tftp (Trivial File Transfer Protocol, 碎文件协议), vat (Visual Audio Tool, 可用于在internet 上进行电
    视电话会议的应用层协议), 以及wb (distributed White Board, 可用于网络会议的应用层协议).

`-t`    在每行输出中不打印时间戳

`-tt`   不对每行输出的时间进行格式处理(nt: 这种格式一眼可能看不出其含义, 如时间戳打印成1261798315)

`-ttt`  tcpdump 输出时, 每两行打印之间会延迟一个段时间(以毫秒为单位)

`-tttt`  在每行打印的时间戳之前添加日期的打印

`-u`    打印出未加密的NFS 句柄(nt: handle可理解为NFS 中使用的文件句柄, 这将包括文件夹和文件夹中的文件)

`-U`

    使得当tcpdump在使用-w 选项时, 其文件写入与包的保存同步.(nt: 即, 当每个数据包被保存时, 它将及时被写入文件中,而不是等文件的输出缓冲已满时才真正写入此文件)

    -U 标志在老版本的libcap库(nt: tcpdump 所依赖的报文捕获库)上不起作用, 因为其中缺乏pcap_cump_flush()函数.

`-v`

    当分析和打印的时候, 产生详细的输出. 比如, 包的生存时间, 标识, 总长度以及IP包的一些选项. 这也会打开一些附加的包完整性检测, 比如对IP或ICMP包头部的校验和.

`-vv`  产生比-v更详细的输出. 比如, NFS回应包中的附加域将会被打印, SMB数据包也会被完全解码.

`-vvv`

     产生比-vv更详细的输出. 比如, telent 时所使用的SB, SE 选项将会被打印, 如果telnet同时使用的是图形界面,
     其相应的图形选项将会以16进制的方式打印出来(nt: telnet 的SB,SE选项含义未知, 另需补充).

`-w`  把包数据直接写入文件而不进行分析和打印输出. 这些包数据可在随后通过-r 选项来重新读入并进行分析和打印.

`-W   filecount`

      此选项与-C 选项配合使用, 这将限制可打开的文件数目, 并且当文件数据超过这里设置的限制时, 依次循环替代之前的文件, 这相当于一个拥有filecount 个文件的文件缓冲池. 同时, 该选项会使得每个文件名的开头会出现足够多并用来占位的0, 这可以方便这些文件被正确的排序.

`-x`

    当分析和打印时, tcpdump 会打印每个包的头部数据, 同时会以16进制打印出每个包的数据(但不包括连接层的头部).总共打印的数据大小不会超过整个数据包的大小与snaplen 中的最小值. 必须要注意的是, 如果高层协议数据没有snaplen 这么长,并且数据链路层(比如, Ethernet层)有填充数据, 则这些填充数据也会被打印.(nt: so for link  layers  that pad, 未能衔接理解和翻译, 需补充 )

`-xx`  tcpdump 会打印每个包的头部数据, 同时会以16进制打印出每个包的数据, 其中包括数据链路层的头部.

`-X`

    当分析和打印时, tcpdump 会打印每个包的头部数据, 同时会以16进制和ASCII码形式打印出每个包的数据(但不包括连接层的头部).这对于分析一些新协议的数据包很方便.

`-XX`

    当分析和打印时, tcpdump 会打印每个包的头部数据, 同时会以16进制和ASCII码形式打印出每个包的数据, 其中包括数据链路层的头部.这对于分析一些新协议的数据包很方便.

`-y datalinktype`

      设置tcpdump 只捕获数据链路层协议类型是datalinktype的数据包

`-Z  user`

      使tcpdump 放弃自己的超级权限(如果以root用户启动tcpdump, tcpdump将会有超级用户权限), 并把当前tcpdump的用户ID设置为user, 组ID设置为user首要所属组的ID(nt: tcpdump 此处可理解为tcpdump 运行之后对应的进程)

      此选项也可在编译的时候被设置为默认打开.(nt: 此时user 的取值未知, 需补充)

##tcpdump的expression

    expression是tcpdump最为有用的高级用法，可以利用它来匹配一些特殊的包。下面介绍一下expression的用法，主要是如何写出符合要求最为严格expression。如果tcpdump中没有expression,那么tcpdump会把网卡上的所有数据包输出，否则会将被expression匹配的包输出。

    expression 由一个或多个[primitives]组成，而[primitives]由一个或多个[qualitifer]加一个id(name)或数字组成，它们的结构如用正则表达式则可表示为：

    expression = ([qualitifer］+(id|number))+

    以此可见，expression是一个复杂的条件表达式，其中[qualitifer］+(id|number)就是一个比较基本条件，qualitifer就表达一些的名称（项，变量），id或number则表示一个值（或常量）。表达式用于决定哪些数据包将被打印. 如果不给定条件表达式, 网络上所有被捕获的包都会被打印,否则, 只有满足条件表达式的数据包被打印.(all packets, 可理解为, 所有被指定接口捕获的数据包).qualitifer有三种形式:

    type  id :对象类型
    dir   id :传输方向
    proto id :协议

**type : ** 修饰符指定id 所代表的对象类型, id可以是名字也可以是数字.**id默认的修饰符为host**.
可选的对象类型有: 

    host      :id是主机,如host foo表示主机是foo
    net       :id是网络,如net 128.3表示网络 128.3
    port      :id是端口,如port 20表示端口 20
    portrange :id是端口范围,如portrange 6000-6008表示端口范围 6000-6008

**dir : ** 修饰符描述id 所对应的传输方向, 即发往id 还是从id 接收（nt: 而id 到底指什么需要看其前面的type 修饰符）.**id默认的修饰符为src 或 dst**.可取的方向为:      

    src         :id是传输源,如src foo表示源主机是foo
    dst         :id是传输目的,如dst net 128.3表示目的网络是128.3
    src or dst  :id是传输源或者传输目的,如
                 如src or dst port ftp-data 表示源或目的端口为ftp-data
    src and dst :id是传输源并且是传输目的,如
                 src or dst port 50 表示源或目的端口为50

对于链路层的协议,比如SLIP(nt: Serial Line InternetProtocol, 串联线路网际网络协议), 以及linux下指定'any' 设备, 并指定'cooked'(nt | rt: cooked 含义未知, 需补充) 抓取类型, 或其他设备类型,可以用'inbound' 和'outbount' 修饰符来指定想要的传输方向.

**proto : ** 修饰符描述id所属的协议. **默认为与相应type匹配的修饰符**,可选的协议有:

        ether   :物理以太网传输协议,如ether src foo
        fddi    :光纤分布数据网传输协议,如arp net 128.3
        tr      :路由跟踪的协议,如tcp port 21
        wlan    :无线局域网协议
        ip/ip6  :通常的TCP/IP协议栈中所使用的ipv4以及ipv6网络层协议
        arp/rarp:地址解析协议,反向地址解析协议
        decnet  :Digital Equipment Corporation开发的, 最早用于PDP-11机器互联的网络协议
        tcp/upd :TCP/IP协议栈中的两个传输层协议,如udp portrange 7000-7009

   对于默认的情况,例如,

     `src foo = (ip or arp or rarp) src foo`: 来自主机foo的ip/arp/rarp协议数据包, 默认type为host)
     `net bar = (ip  or  arp  or rarp) net bar`: 来自或发往bar网络的ip/arp/rarp协议数据包
     `port 53 = (tcp or udp) port 53' : 发送或接收端口为53的tcp/udp协议数据包 

   注:

   由于tcpdump 直接通过数据链路层的 BSD 数据包过滤器或 DLPI(datalink provider interface, 数据链层提供者接口)来直接获得网络数据包, 其可抓取的数据包可涵盖上层的各种协议, 包括arp, rarp, icmp(因特网控制报文协议),ip, ip6, tcp, udp, sctp(流控制传输协议).

   fddi(Fiber Distributed Data Interface) 实际上与ether含义一样: tcpdump 会把他们当作一种指定网络接口上的数据链路层协议. 如同ehter网(以太网), FDDI 的头部通常也会有源, 目的, 以及包类型, 从而可以像ether网数据包一样对这些域进行过滤. 此外, FDDI 头部还有其他的域, 但不能被放到表达式中用来过滤

   同样, tr 和 wlan 也和 ether 含义一致, 上一段对fddi 的描述同样适用于tr(Token Ring) 和wlan(802.11 wireless LAN)的头部. 对于802.11协议数据包的头部, 目的域称为DA, 源域称为SA;而其中的 BSSID, RA, TA 域(具体含义需补充)不会被检测(不能被用于包过虑表达式中).

   除以上所描述的表达元(primitive)， 还有其他形式的表达式, 并且与上述表达式格式不同. 比如: gateway, broadcast, less, greater以及算术表达式(其中每一个都算一种新的表达式). 下面将会对这些表达式进行说明.


##tcpdump 原语

1.匹配使用host作为网关的数据包，即数据报中mac地址（源或目的）为host，但IP报的源和目的地址不是host的数据包。

    gateway host

2.匹配IPv4/v6地址为net网络的数据报。

    dst net net
    src net net
    net net
    net net mask netmask:仅对IPv4数据包有效，如net 192.168.0.0 mask 255.255.0.0
    net net/len :只对IPv4数据包有效，如net 192.168.0.0/16

其中net可以为192.168.0.0或192.168这两种形式。如net 192.168 或net 192.168.0.0

3.匹配长度

    less length    :匹配长度少于等于length的报文。
    greater length :匹配长度大于等于length的报文。
                    例`tcpdump 'ip[2:2] > 576'`打印IP包长超过576字节的网络包 

4.匹配字段

    ip protochain protocol  :匹配ip报文中protocol字段值为protocol的报文
    ip6 protochain protocol :匹配ipv6报文中protocol字段值为protocol的报文

如tcpdump 'ip protochain 6 匹配ipv4网络中的TCP报文，与tcpdump 'ip && tcp'用法一样，这里的&&连接两个primitive。6是TCP协议在IP报文中的编号。

5.匹配广播

    ether broadcast :匹配以太网广播报文
    ether multicast :匹配以太网多播报文
    ip broadcast :匹配IPv4的广播报文。也即IP地址中主机号为全0或全1的IPv4报文。
    ip multicast :匹配IPv4多播报文，也就是IP地址为多播地址的报文。
    ip6 multicast:匹配IPv6多播报文，即IP地址为多播地址的报文。

6.匹配vlan

    vlan vlan_id :匹配为vlan报文 ，且vlan号为vlan_id的报文 

    除此之外,表达式之间还可以通过关键字and, or 以及 not 进行连接, 从而可组成比较复杂的条件表达式. 比如,`host foo and not port ftp and not port ftp-data'(其过滤条件可理解为, 数据包的主机为foo,并且端口不是ftp(端口21) 和ftp-data(端口20,**常用端口和名字的对应可在linux 系统中的/etc/service 文件中找到**)

   为了表示方便, 同样的修饰符可以被省略, 如`tcp dst port ftp or ftp-data or domain` 与以下的表达式含义相同`tcp dst port ftp or tcp dst port ftp-data or tcp dst port domain`.(其过滤条件可理解为,包的协议为tcp, 目的端口为ftp 或 ftp-data 或 domain(端口53) ).

   借助括号以及相应操作符,可把表达元组合在一起使用(由于括号是shell的特殊字符, 所以在shell脚本或终端中使用时必须对括号进行转义, 即'(' 与')'需要分别表达成'\(' 与 '\)').


有效的操作符有:

     否定操作 (`!' 或 `not')
     与操作(`&&' 或 `and')
     或操作(`||' 或 `or')

   **否定操作符的优先级别最高. 与操作和或操作优先级别相同, 并且二者的结合顺序是从左到右.** 要注意的是, 表达'与操作'时,需要显式写出'and'操作符, 而不只是把前后表达元并列放置(二者中间的'and' 操作符不可省略).

###最重要形式：`expr relop expr`
**注:对此处内容的掌握需要你熟悉相关协议**

    relop: 表示关系操作符，可以为>, < ,>=,<=, =, !=之一，
    expr : 是一个算术表达式，由整数组成和二元运算符（＋，－，＊，/，＆，|, <<, >>)，
           长度操作，报文数据访问子。同时所有的整数都是无符号的，即0x80000000 和
            0xffffffff > 0。为了访问报文中的数据，可使用如下方式：

           proto [ expr : size ]
           proto表示该问的报文，expr的结果表示该报文的偏移，size为可选的，表示从
           expr偏移量起的szie个字节，整个表达式为proto报文中,expr起的szie字节的
           内容（无符号整数）

下面是expr relop expr这种形式primitive的例子：

`ether[0] & 1 !=0`: ether报文中第0个bit为1，即以太网广播或组播的primtive。

`ip[0] = 4` : ip报文中的第一个字节为version，即匹配IPv4的报文，

`tcp[13] = 2` :

    匹配一个syn报文，因为tcp的标志位为TCP报文的第13个字节，而syn在这个字节的低1位，故匹配只有syn标志的报文,上述条件是可满要求的，并且比较严格。

`icmp[0]=8`:

    匹配ping命令的请求报文，因为icmp报文的第0字符表示类型，当类型值为8时表示为回显示请求。

    对于TCP和ICMP中常用的字节，如TCP中的标志位，ICMP中的类型，这个些偏移量有时会忘记。不过tcpdump为你提供更方便的用法，你不用记位这些数字，用字符就可以代替了.

    对于ICMP报文，类型字节可以icmptype来表示它的偏称量，上面的primitive可改为'icmp[icmptype] =8'，如果8也记不住怎么办？tcpdump还为该字节的值也提供了字符表示，如'icmp[icmptype] = icmp-echo'。

下面是tcpdump提供的字符偏移量：

    `icmptype` ：表示icmp报文中类弄字节的偏移量

    `icmpcode` :表示icmp报文中编码字节的偏移量

    `tcpflags` :表示TCP报文中标志位字节的偏移量

此外，还提供了很多值来对应上面的偏移字节：

ICMP中类型字节的值可以是：

    icmp-echoreply, icmp-unreach, icmp-sourcequench, icmp-redi‐rect, icmp-echo,
    icmp-routeradvert, icmp-routersolicit,

    icmp-timxceed, icmp-paramprob, icmp-tstamp, icmp-tstam‐preply, icmp-ireq,
    icmp-ireqreply, icmp-maskreq, icmp-maskreply.

TCP中标志位字节的值可以是：

    tcp-fin, tcp-syn, tcp-rst, tcp-push, tcp-ack, tcp-urg.

通过上面的字符表示，我们可以写出下面的primitive

    `tcp[tcpflags] = tcp-syn`: 匹配只有syn标志设置为1的 tcp报文
    `tcp[tcpflags] & (tcp-syn |tcp-ack |tcp-fin) !=0` 匹配含有syn，或ack或fin标志位的TCP报文

对于IP报文，没有提供字符支持，如果想匹配更细的条件，直接使用数字指字偏移量就可以了，不过要对IP报文有更深入的了解才可以。

学会写primitive后，expression就是小菜一碟了，由一个或多个primitive组成，并且逻辑连接符组成即可：

    tcpdump ‘host 192.168.240.91 && icmp[icmptype] = icmp-echo’
    tcpdump ‘host 192.168.1.100 && vrrp’
    tcpdump 'ether src 00:00:00:00:00:02 && ether[0] & 1 !=0'
    tcpdump -i <interface> "tcp[tcpflags] & (tcp-syn) != 0"
    tcpdump -i <interface> "tcp[tcpflags] & (tcp-ack) != 0"
    tcpdump -i <interface> "tcp[tcpflags] & (tcp-fin) != 0"
    tcpdump -r <interface> "tcp[tcpflags] & (tcp-syn|tcp-ack) != 0"

让你随心所欲地使用tcpdump，将不用再从复杂的输出中去挑报文了！

**NOTE**

    如果一个标识符前没有关键字, 则表达式的解析过程中最近用过的关键字(往往也是从左往右距离标识符最近的关键字)将被使用.比如,`not host vs and ace`是`not host vs and host ace` 的精简,而不是not (host vs or ace).(前两者表示, 所需数据包不是来自或发往host vs, 而是来自或发往ace.而后者表示数据包只要不是来自或发往vs或ac都符合要求)

    整个条件表达式可以被当作一个单独的字符串参数也可以被当作空格分割的多个参数传入tcpdump, 后者更方便些. 通常, 如果表达式中包含元字符(如正则表达式中的'*', '.'以及shell中的'('等字符)，最好还是使用单独字符串的方式传入. 这时,整个表达式需要被单引号括起来. 多参数的传入方式中, 所有参数最终还是被空格串联在一起, 作为一个字符串被解析.


#tcpdump输出格式

   总的的输出格式为：系统时间 来源主机.端口 > 目标主机.端口 数据包参数 

   tcpdump 对截获的数据并没有进行彻底解码，数据包内的大部分内容是使用十六进制的形式直接打印输出的。显然这不利于分析网络故障，通常的解决办法是先使用带-w参数的tcpdump 截获数据并保存到文件中，然后再使用其他程序(如Wireshark)进行解码分析。当然也应该定义过滤规则，以避免捕获的数据包填满整个硬盘。

###数据链路层头信息

    tcpdump --e host ICE

ICE 是一台装有linux的主机。它的MAC地址是0：90：27：58：AF：1A H219是一台装有Solaris的SUN工作站。它的MAC地址是8：0：20：79：5B：46；输出结果如下所示：

    21:50:12.847509 eth0 < 8:0:20:79:5b:46 0:90:27:58:af:1a ip 60:
         h219.33357 > ICE. telne t 0:0(0) ack 22535 win 8760 (DF)

21：50：12是显示的时间， 847509是ID号，eth0 <表示从网络接口eth0接收该分组， eth0 >表示从网络接口设备发送分组， 8:0:20:79:5b:46是主机H219的MAC地址， 它表明是从源地址H219发来的分组. 0:90:27:58:af:1a是主机ICE的MAC地址， 表示该分组的目的地址是ICE。 ip 是表明该分组是IP分组，60 是分组的长度， h219.33357 > ICE. telnet 表明该分组是从主机H219的33357端口发往主机ICE的 TELNET(23)端口。 ack 22535 表明对序列号是222535的包进行响应。 win 8760表明发 送窗口的大小是8760。

###ARP/RARP 数据包

    tcpdump arp

    22:32:42.802509 eth0 > arp who-has route tell ICE (0:90:27:58:af:1a)
    22:32:42.802902 eth0 < arp reply route is-at 0:90:27:12:10:66 (0:90:27:58:af:1a)

22:32:42是时间戳， 802509是ID号， eth0 >表明从主机发出该分组，arp表明是ARP请求包， who-has route tell ICE表明是主机ICE请求主机route的MAC地址。0:90:27:58:af:1a是主机 ICE的MAC地址。

###UDP包的输出信息

tcpdump捕获的UDP包的一般输出信息是：

    route.port1 > ICE.port2: udp lenth

UDP十分简单，上面的输出行表明从主机route的port1端口发出的一个UDP报文到主机ICE的port2端口，类型是UDP，包的长度是lenth。


###TCP 数据包

**TCP Header Format**

    0                   1                   2                   3   
    0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |          Source Port          |       Destination Port        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                        Sequence Number                        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Acknowledgment Number                      |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |  Data |           |U|A|P|R|S|F|                               |
   | Offset| Reserved  |R|C|S|S|Y|I|            Window             |
   |       |           |G|K|H|T|N|N|                               |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |           Checksum            |         Urgent Pointer        |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                    Options                    |    Padding    |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   |                             data                              |
   +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+


通常tcpdump对tcp数据包的显示格式如下:
src > dst: flags data-seqno ack window urgent options

    src 和 dst 是源和目的IP地址以及相应的端口. 
    flags 标志由S(SYN), F(FIN), P(PUSH, R(RST),W(ECN CWT)或者 E(ECN-Echo)组成,单独一个'.'表示没有flags标识. 
    数据段顺序号(Data-seqno)描述了此包中数据所对应序列号空间中的一个位置(整个数据被分段,每段有一个顺序号, 所有的顺序号构成一个序列号空间).
    Ack 描述的是同一个连接,同一个方向,下一个本端应该接收的(对方应该发送的)数据片段的顺序号.
    Window是本端可用的数据接收缓冲区的大小(也是对方发送数据时需根据这个大小来组织数据).
    Urg(urgent) 表示数据包中有紧急的数据. 
    options 描述了tcp的一些选项, 这些选项都用尖括号来表示(如 <mss 1024>).
    (注:src, dst 和 flags 这三个域总是会被显示. 其他域的显示与否依赖于tcp协议头里的信息.)

练习: 自己组织命令,抓与某一ip地址的tcp连接过程,并与上面解释对比.

###抓取带有特殊标志的的TCP包(如SYN-ACK标志, URG-ACK标志等).

在TCP的头部中, 有8比特(bit)用作控制位区域, 其取值为:
CWR | ECE | URG | ACK | PSH | RST | SYN | FIN

现假设我们想要监控建立一个TCP连接整个过程中所产生的数据包. 可回忆如下:TCP使用3次握手协议来建立一个新的连接; 其与此三次握手
连接顺序对应，并带有相应TCP控制标志的数据包如下:

    1) 连接发起方(nt:Caller)发送SYN标志的数据包
    2) 接收方(nt:Recipient)用带有SYN和ACK标志的数据包进行回应
    3) 发起方收到接收方回应后再发送带有ACK标志的数据包进行回应

一个TCP头部,在不包含选项数据的情况下通常占用20个字节. 第一行包含0到3编号的字节,
第二行包含编号4-7的字节.

如果编号从0开始算, TCP控制标志位于13字节.

    0 7| 15| 23| 31
    ----------------|---------------|---------------|----------------
    | HL | rsvd |C|E|U|A|P|R|S|F| window size |
    ----------------|---------------|---------------|----------------
    | | 13th octet | | |

    让我们仔细看看编号13的字节:

    | |
    |---------------|
    |C|E|U|A|P|R|S|F|
    |---------------|
    |7 5 3 0|


这里有我们感兴趣的控制标志位. 从右往左这些位被依次编号为0到7, 从而 PSH位在3号, 而URG位在5号.

提醒一下自己, 我们只是要得到包含SYN标志的数据包. 让我们看看在一个包的包头中, 如果SYN位被设置, 到底
在13号字节发生了什么:

    |C|E|U|A|P|R|S|F|
    |---------------|
    |0 0 0 0 0 0 1 0|
    |---------------|
    |7 6 5 4 3 2 1 0|

在控制段的数据中, 只有比特1(bit number 1)被置位.

假设编号为13的字节是一个8位的无符号字符型,并且按照网络字节号排序(nt:对于一个字节来说，网络字节序等同于主机字节序), 其二进制值如下所示:

    00000010

并且其10进制值为:

    0*2^7 + 0*2^6 + 0*2^5 + 0*2^4 + 0*2^3 + 0*2^2 + 1*2^1 + 0*2^0 = 2

接近目标了, 因为我们已经知道, 如果数据包头部中的SYN被置位, 那么头部中的第13个字节的值为2(nt: 按照网络序, 即大头方式, 最重要的字节在前面(在前面,即该字节实际内存地址比较小, 最重要的字节,指数学表示中数的高位, 如356中的3) ).表达为tcpdump能理解的关系式就是: tcp[13] 2

从而我们可以把此关系式当作tcpdump的过滤条件, 目标就是监控只含有SYN标志的数据包:

    tcpdump -i et0 tcp[13] 2

这个表达式是说"让TCP数据包的第13个字节拥有值2吧", 这也是我们想要的结果.


现在, 假设我们需要抓取带SYN标志的数据包, 而忽略它是否包含其他标志. 让我们来看看当一个含有
SYN-ACK的数据包(nt:SYN 和 ACK 标志都有), 来到时发生了什么:

    |C|E|U|A|P|R|S|F|
    |---------------|
    |0 0 0 1 0 0 1 0|
    |---------------|
    |7 6 5 4 3 2 1 0|

13号字节的1号和4号位被置位, 其二进制的值为:
00010010

转换成十进制就是:

    0*2^7 + 0*2^6 + 0*2^5 + 1*2^4 + 0*2^3 + 0*2^2 + 1*2^1 + 0*2 = 18

现在, 却不能只用'tcp[13] 18'作为tcpdump的过滤表达式, 因为这将导致只选择含有SYN-ACK标志的数据包, 其他的都被丢弃.
提醒一下自己, 我们的目标是: 只要包的SYN标志被设置就行, 其他的标志我们不理会.

为了达到我们的目标, 我们需要把13号字节的二进制值与其他的一个数做AND操作来得到SYN比特位的值. 目标是:只要SYN 被设置
就行, 于是我们就把她与上13号字节的SYN值(00000010).

    00010010 SYN-ACK 00000010 SYN
    AND 00000010 (we want SYN) AND 00000010 (we want SYN)
    -------- --------
    = 00000010 = 00000010

我们可以发现, 不管包的ACK或其他标志是否被设置, 以上的AND操作都会给我们相同的值, 其10进制表达就是2(2进制表达就是00000010).
从而我们知道, 对于带有SYN标志的数据包, 以下的表达式的结果总是真(true):

    ( ( value of octet 13 ) AND ( 2 ) ) ( 2 ) (nt: value of octet 13, 即13号字节的值)

灵感随之而来, 我们于是得到了如下的tcpdump 的过滤表达式

    tcpdump -i xl0 'tcp[13] & 2 2'

注意, 单引号或反斜杆(nt: 这里用的是单引号)不能省略, 这可以防止shell对&的解释或替换.


###tcpdump 与wireshark

   如果要用wireshark分析数据：
        tcpdump -i eth0 -c 100 -s 0 -w /home/data.pcap 

   直接使用wireshark /home/data.pcap即可


##实用命令实例

###监视指定主机的数据包

1.截获所有210.27.48.1 的主机收到的和发出的所有的数据包:

    tcpdump host 210.27.48.1

2.打印helios 与 hot 或者与 ace 之间通信的数据包

    tcpdump host helios and \( hot or ace \)

3.截获主机210.27.48.1 和主机210.27.48.2 或210.27.48.3的通信

    tcpdump host 210.27.48.1 and \ (210.27.48.2 or 210.27.48.3 \) 

4.打印ace与任何其他主机之间通信的IP 数据包, 但不包括与helios之间的数据包.

    tcpdump ip host ace and not helios

5.如果想要获取主机210.27.48.1除了和主机210.27.48.2之外所有主机通信的ip包，使用命令：

    tcpdump ip host 210.27.48.1 and ! 210.27.48.2

6.截获主机hostname发送的所有通过eth0的数据

    tcpdump -i eth0 src host hostname

7.监视所有通过eth0送到主机hostname的数据包

    tcpdump -i eth0 dst host hostname

8.过滤源主机物理地址为XXX的报头：

    tcpdump ether src 00:50:04:BA:9B and dst……

（为什么ether src后面没有host或者net？物理地址当然不可能有网络喽）。 

9.我们还可以监视通过指定网关的数据包：

    tcpdump -i eth0 gateway Gatewayname

###监视指定主机和端口的数据包

1.如果想要获取主机210.27.48.1接收或发出的telnet包，使用如下命令

    tcpdump tcp port 23 host 210.27.48.1

2.对本机的udp 123 端口进行监视 123 为ntp的服务端口

    tcpdump udp port 123 

3.如何使用 tcpdump 监听 (1)來自 eth0 接口 (2)协议为端口 port 22 ，(3)目标来源是 192.168.1.100 的封包资料？ 

    tcpdump -i eth0 -nn 'port 22 and src host 192.168.1.100' 

4.如果我们只需要列出送到80端口的数据包，用dst port 80；如果我们只希望看到返回80端口的数据包，用src port 80。 
    
    tcpdump –i eth0 host hostname and dst port 80  目的端口是80 
    tcpdump –i eth0 host hostname and src port 80  源端口是80 
    80端口一般是提供http的服务的主机
     
###监视指定网络的数据包

1.打印所有通过网关snup的ftp数据包(注意, 表达式被单引号括起来了, 这可以防止shell对其中的括号进行错误解析)

    tcpdump 'gateway snup and (port ftp or ftp-data)'

2.打印所有源地址或目标地址不是本地主机的IP数据包(localnet 实际使用时要真正替换成本地网络的名字)

    tcpdump ip and not net localnet
    
3.我们还可以监视通过指定网关的数据包： 

    tcpdump -i eth0 gateway Gatewayname 
    
###监视指定协议的数据包

1.打印TCP会话中的的开始和结束数据包, 并且数据包的源或目的不是本地网络上的主机.(localnet, 实际使用时要真正替换成本地网络的名字))

    tcpdump 'tcp[tcpflags] & (tcp-syn|tcp-fin) != 0 and not src and dst net localnet'

2.打印所有源或目的端口是80, 网络层协议为IPv4, 并且含有数据,而不是SYN,FIN以及ACK-only等不含数据的数据包.(ipv6的版本的表达式可做练习)

    tcpdump 'tcp port 80 and (((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0)'

(可理解为, ip[2:2]表示整个ip数据包的长度, (ip[0]&0xf)<<2)表示ip数据包包头的长度(ip[0]&0xf代表包中的IHL域, 而此域的单位为32bit, 要换算成字节数需要乘以4,　即左移2.　(tcp[12]&0xf0)>>4 表示tcp头的长度, 此域的单位也是32bit,　换算成比特数为 ((tcp[12]&0xf0) >> 4)　<<　２,　
即 ((tcp[12]&0xf0)>>2).　((ip[2:2] - ((ip[0]&0xf)<<2)) - ((tcp[12]&0xf0)>>2)) != 0　表示: 整个ip数据包的长度减去ip头的长度,再减去
tcp头的长度不为0, 这就意味着, ip数据包中确实是有数据.对于ipv6版本只需考虑ipv6头中的'Payload Length' 与 'tcp头的长度'的差值, 并且其中表达方式'ip[]'需换成'ip6[]'.)

3.打印长度超过576字节, 并且网关地址是snup的IP数据包

    tcpdump 'gateway snup and ip[2:2] > 576'

4.打印所有IP层广播或多播的数据包， 但不是物理以太网层的广播或多播数据报

    tcpdump 'ether[0] & 1 = 0 and ip[16] >= 224'

5.打印除'echo request'或者'echo reply'类型以外的ICMP数据包( 比如,需要打印所有非ping 程序产生的数据包时可用到此表达式.('echo reuqest' 与 'echo reply' 这两种类型的ICMP数据包通常由ping程序产生))

    tcpdump 'icmp[icmptype] != icmp-echo and icmp[icmptype] != icmp-echoreply'

6.使用tcpdump抓取HTTP包.(0x4745 为"GET"前两个字母"GE",0x4854 为"HTTP"前两个字母"HT")

    tcpdump  -XvvennSs 0 -i eth0 tcp[20:2]=0x4745 or tcp[20:2]=0x4854
    
7.和baidu.com之间建立TCP三次握手中第一个网络包，即带有SYN标记位的网络包，另外，目的主机不能是qiyi.com
   
    tcpdump 'tcp[tcpflags] & tcp-syn != 0 and not dst host qiyi.com'
    
 
8.tcpdump tcp -i eth1 -t -s 0 -c 100 and dst port ! 22 and src net 192.168.1.0/24 -w ./target.cap

    (1)tcp: ip icmp arp rarp 和 tcp、udp、icmp这些选项等都要放到第一个参数的位置，用来过滤数据报的类型
    (2)-i eth1 : 只抓经过接口eth1的包
    (3)-t : 不显示时间戳
    (4)-s 0 : 抓取数据包时默认抓取长度为68字节。加上-S 0 后可以抓到完整的数据包
    (5)-c 100 : 只抓取100个数据包
    (6)dst port ! 22 : 不抓取目标端口是22的数据包
    (7)src net 192.168.1.0/24 : 数据包的源网络地址为192.168.1.0/24
    (8)-w ./target.cap : 保存成cap文件，方便用ethereal(即wireshark)分析

9.打印广播包或多播包，同时数据链路层不是通过以太网媒介进行的。

    tcpdump 'ether[0] & 1 = 0 and ip[16] >= 224'


##协议分析

####范例一：以 IP 与 port number 捉下 eth0 这个网路卡上的封包，持续 3 秒 

[root@linux ~]# tcpdump -i eth0 -nn 

    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode 
    listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes 
    01:33:40.41 IP 192.168.1.100.22 > 192.168.1.11.1190: P 116:232(116) ack 1 win 9648 
    01:33:40.41 IP 192.168.1.100.22 > 192.168.1.11.1190: P 232:364(132) ack 1 win 9648 
    <==按下 [ctrl]-c 之后结束 
    6680 packets captured              <==捉下来的封包数量 
    14250 packets received by filter   <==由过滤所得的总封包数量 
    7512 packets dropped by kernel     <==被核心所丢弃的封包 

如果你是第一次看 tcpdump 的 man page 时，肯定一个头两个大，因为 tcpdump 几乎都是分析封包的表头资料，使用者如果没有简易的网路封包基础，要看懂粉难呐！ 所以，至少您得要回到网路基础里面去将 TCP 封包的表头资料理解理解才好啊！ ^_^！至于那个范例一所产生的输出范例中，我们可以约略区分为数个栏位， 我们以范例一当中那个特殊字体行来说明一下： 

    01:33:40.41：这个是此封包被撷取的时间，时:分:秒的单位； 
    IP：透过的通讯协定是 IP ； 
    192.168.1.100.22 > ：传送端是 192.168.1.100 这个 IP，而传送的 port number 为 22，您必须要瞭解的是，那个大于 (>) 的符号指的是封包的传输方向喔！ 
    192.168.1.11.1190：接收端的 IP 是 192.168.1.11， 且该主机开启 port 1190 来接收； 
    P 116:232(116)：这个封包带有 PUSH 的资料传输标志， 且传输的资料为整体资料的 116~232 byte，所以这个封包带有 116 bytes 的资料量； 
    ack 1 win 9648：ACK与 Window size 的相关资料。 

最简单的说法，就是该封包是由 192.168.1.100 传到 192.168.1.11，透过的 port 是由 22 到 1190 ， 且带有 116 bytes 的资料量，使用的是 PUSH 的旗标，而不是 SYN 之类的主动连线标志。 呵呵！不容易看的懂吧！所以说，上头才讲请务必到 TCP 表头资料的部分去瞧一瞧的啊！ 

再来，一个网路状态很忙的主机上面，你想要取得某部主机对你连线的封包资料时， 使用 tcpdump 配合管道命令与正规表达式也可以，不过，毕竟不好捉取！ 我们可以透过 tcpdump 的表示法功能，就能够轻易的将所需要的资料独立的取出来。 在上面的范例一当中，我们仅针对 eth0 做监听，所以整个 eth0 介面上面的资料都会被显示到萤幕上， 不好分析啊！那麽我们可以简化吗？例如只取出 port 21 的连线封包，可以这样做： 

[root@linux ~]# tcpdump -i eth0 -nn port 21 

    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode 
    listening on eth0, link-type EN10MB (Ethernet), capture size 96 bytes 
    01:54:37.96 IP 192.168.1.11.1240 > 192.168.1.100.21: . ack 1 win 65535 
    01:54:37.96 IP 192.168.1.100.21 > 192.168.1.11.1240: P 1:21(20) ack 1 win 5840 
    01:54:38.12 IP 192.168.1.11.1240 > 192.168.1.100.21: . ack 21 win 65515 
    01:54:42.79 IP 192.168.1.11.1240 > 192.168.1.100.21: P 1:17(16) ack 21 win 65515 
    01:54:42.79 IP 192.168.1.100.21 > 192.168.1.11.1240: . ack 17 win 5840 
    01:54:42.79 IP 192.168.1.100.21 > 192.168.1.11.1240: P 21:55(34) ack 17 win 5840 

瞧！这样就仅抓出 port 21 的包，如果仔细看的话，你会发现封包的传递都是双向的， client 端发出请求而 server 端则予以应答，所以，当然是有去有回啊！ 而我们也就可以经过这个封包的流向来了解到封包运作的过程。 举例来说： 
我们先在一个终端输入 tcpdump -i lo -nn  的监听， 
再另开一个终端机视窗来对本机 (127.0.0.1) 登入 ssh localhost 
那么输出的结果会是如何？ 

[root@linux ~]# tcpdump -i lo -nn 

    1 tcpdump: verbose output suppressed, use -v or -vv for full protocol decode 
    2 listening on lo, link-type EN10MB (Ethernet), capture size 96 bytes 
    3 11:02:54.253777 IP 127.0.0.1.32936 > 127.0.0.1.22: S 933696132:933696132(0) 
       win 32767 <mss 16396,sackOK,timestamp 236681316 0,nop,wscale 2> 
    4 11:02:54.253831 IP 127.0.0.1.22 > 127.0.0.1.32936: S 920046702:920046702(0) 
       ack 933696133 win 32767 <mss 16396,sackOK,timestamp 236681316 236681316,nop, 
       wscale 2> 
    5 11:02:54.253871 IP 127.0.0.1.32936 > 127.0.0.1.22: . ack 1 win 8192 <nop, 
       nop,timestamp 236681316 236681316> 
    6 11:02:54.272124 IP 127.0.0.1.22 > 127.0.0.1.32936: P 1:23(22) ack 1 win 8192 
       <nop,nop,timestamp 236681334 236681316> 
    7 11:02:54.272375 IP 127.0.0.1.32936 > 127.0.0.1.22: . ack 23 win 8192 <nop, 
       nop,timestamp 236681334 236681334> 

上表显示的头两行是 tcpdump 的基本说明，然后： 

    第 3 行显示的是『来自 client 端，带有 SYN 主动连线的封包』， 
    第 4 行显示的是『来自 server 端，除了回应 client 端之外(ACK)，还带有 SYN 主动连线的标志； 
    第 5 行则显示 client 端回应 server 确定连线建立 (ACK) 
    第 6 行以后则开始进入资料传输的步骤。 

从第 3-5 行的流程来看，熟不熟悉啊？没错！那就是 三次握手的基础流程啦！够有趣吧！ 不过 tcpdump 之所以被称为骇客软体之一可不止上头介绍的功能呐！ 上面介绍的功能可以用来作为我们主机的封包连线与传输的流程分析， 这将有助于我们了解到封包的运作，同时了解到主机的防火墙设定规则是否有需要修订的地方。 

更神奇的使用要来啦！如果我们使用 tcpdump 在 router 上面监听**明码**的传输资料时， 例如 FTP 传输协定，你觉得会发生什麽问题呢？ 我们先在主机端下达` tcpdump -i lo port 21 -nn -X `然后再以 ftp 登入本机，并输入帐号与密码， 结果你就可以发现如下的状况： 

[root@linux ~]# tcpdump -i lo -nn -X 'port 21' 

    0x0000:  4500 0048 2a28 4000 4006 1286 7f00 0001  E..H*(@.@....... 
    0x0010:  7f00 0001 0015 80ab 8355 2149 835c d825  .........U!I./.% 
    0x0020:  8018 2000 fe3c 0000 0101 080a 0e2e 0b67  .....<.........g 
    0x0030:  0e2e 0b61 3232 3020 2876 7346 5450 6420  ...a220.(vsFTPd. 
    0x0040:  322e 302e 3129 0d0a                      2.0.1).. 

    0x0000:  4510 0041 d34b 4000 4006 6959 7f00 0001  E..A.K@.@.iY.... 
    0x0010:  7f00 0001 80ab 0015 835c d825 8355 215d  ........./.%.U!] 
    0x0020:  8018 2000 fe35 0000 0101 080a 0e2e 1b37  .....5.........7 
    0x0030:  0e2e 0b67 5553 4552 2064 6d74 7361 690d  ...gUSER.dmtsai. 
    0x0040:  0a                                       . 

    0x0000:  4510 004a d34f 4000 4006 694c 7f00 0001  E..J.O@.@.iL.... 
    0x0010:  7f00 0001 80ab 0015 835c d832 8355 217f  ........./.2.U!. 
    0x0020:  8018 2000 fe3e 0000 0101 080a 0e2e 3227  .....>........2' 
    0x0030:  0e2e 1b38 5041 5353 206d 7970 6173 7377  ...8PASS.mypassw 
    0x0040:  6f72 6469 7379 6f75 0d0a                 ordisyou.. 

上面的输出结果已经被简化过了，你必须要自行在你的输出结果当中搜寻相关的字串才行。 从上面输出结果的特殊字体中，我们可以发现『该 FTP 软体使用的是 vsftpd ，并且使用者输入 dmtsai 这个帐号名称，且密码是 mypasswordisyou』 嘿嘿！你说可不可怕啊！如果使用的是明码的方式来传输你的网路资料？ 所以我们才常常在讲啊，网路是很不安全


另外你得了解，为了让网络接口可以让 tcpdump 监听，所以执行 tcpdump 时网络接口会启动在 **错乱模式(promiscuous)**，所以你会在 `/var/log/messages` 里面面看到很多的警告讯息， 通知你说你的网路卡被设定成为错乱模式！别担心，那是正常的。 至于更多的应用，请参考 man tcpdump！ 




#附录
##一次UDP抓包分析

# tcpdump -i eth0 -nn -X 'port 53' -c 1

    tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
    listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes
    19:48:33.285838 IP 116.255.245.206.47940 > 8.8.8.8.53: 22768+ A? www.baidu.com. (31)
    0x0000: 4500 003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01 du.com.....
    1 packets captured
    1 packets received by filter
    0 packets dropped by kernel

    “tcpdump: verbose output suppressed, use -v or -vv for full protocol decode”

这一行，是一句贴心的提醒，简单易懂，就是说你的命令里没有用到-v和-vv，如果希望看到更全的输出内容，可以使用这两个选项。如果你有兴趣，可以试试，看看加上-vv后，到底输出内容里会多些什么。

    “listening on eth0, link-type EN10MB (Ethernet), capture size 65535 bytes”

这一句表示我们监听的是通过eth0这个NIC设备的网络包，且它的链路层是基于以太网的，要抓的包大小限制是65535字节。包大小限制值可以通过-s选项来设置，如果你要追求高性能，建议把这个值调低，这样可以有效避免在大流量情况下的丢包现象。


    “19:48:33.285838 IP 116.255.245.206.47940 > 8.8.8.8.53: 22768+ A? www.baidu.com. (31)”

“19:48:33.285838”，分别对应着这个包被抓到的“时”、“分”、“秒”、“微妙”。

    “IP”，表示这个包在网络层是IP包。

“116.255.245.206.47940”，表示这个包的源IP为116.255.245.206，源端口为47940。

    “>”，这个大于号表示数据包的传输方向。

“8.8.8.8.53“，表示这个包要发向的目的端IP是8.8.8.8，目标端口为53，也就是我们熟知的DNS服务端口。

    “22768+ A? www.baidu.com. (31)“，这是DNS协议的内容，即请求www.baidu.com的A纪录。

    0x0000: 4500 003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

接下来便是IP包的内容了，是除去了以太网之后剩下的内容，其中左侧红色字体部分是十六进制内容，右侧天蓝色字体部分是相应的ASCII码内容。
如果想看懂上面这些十六进制数字，前提就是大家要对IP、TCP/UDP的包格式很熟悉才可以。

    0x0000: 【45】00 003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

如上，最外层是IP数据包，最开始的一个字节（8bit）中，前4bit表示IP的版本，此处为4，表示这是一个IPv4版本的IP包；后4bit表示这IP包的首部长度，此处的数字是5，由于单位是“4字节”，因此可以计算得出这个IP包的首部长度是固定的20字节。如下绿色字体部分都是IP数据包的首部部分：

    0x0000: 【4500 003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808】 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

如下所示，在IP版本和首部长度之后，接下来的一个字节（8bit）是“00”，这是IP协议的服务类型域（TOS），由于已经很少使用，因此此处被置为00。

    0x0000: 45【00】003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

如下所示，后面的2字节（16bit）是“003b”，表示整个IP包的总长度（首部长度+数据长度），单位是字节，因此可以知道这个IP包的总长度是59字节（0x3b需要转换为十进制）。你可以数数下面的红色部分，应该是正好59个字节

    0x0000: 4500 【003b】 c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

再向下的2字节（16bit）是“标识域”，如果IP包的大小超过了数据链路层的MTU限制，就需要对IP包进行分拆，此时就要用这个域来表示哪些包在分拆前是同一组的。此处的标识域值为0xc341。

    0x0000: 4500 003b 【c341】 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

再继续向向下看，便是3bit的标志位，最高位为保留位，中间一位为DF（don’t fragment），最低位为MF（more fragments），可以看到这三位是用来控制IP拆包后的组装所用。由于此包没有拆包，因此这三位都被置为0，如下所示：

    0x0000: 4500 003b c341 【0】000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

和上面的标志位配合的是紧接着的13bit的片便宜，但由于本包没有拆包，因此此域也无用，因此为0。

    0x0000: 4500 003b c341 【0000】 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

如下，紧接着是8bit的TTL（Time To Live，即生存周期），此包的值为0×40，换算成十进制是64，这表明这个网络包，如果经过了超过64个中间路由节点，则认为目的地不可达，中间路由器会将此包抛弃掉。

    0x0000: 4500 003b c341 0000 【40】11 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

继续的8bit为协议域，用于指代上一层协议类型。此处的值为0×11，对应十进制的17，而17是UDP协议的代号，因此可以知道这个网络包所用的传输层协议是UDP，而非TCP（TCP的协议号是6、TCMP的协议号是1）。

    0x0000: 4500 003b c341 0000 40【11】 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

接下来的2个字节表示IP首部校验和，此处计算出来的结果是3c93。

    0x0000: 4500 003b c341 0000 4011 【3c93】 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

如下，再往下就是大家非常熟悉的4字节的IP源地址，即“74 ff f5 ce”，转换成IP地址则为116.255.245.206。

    0x0000: 4500 003b c341 0000 4011 3c93 【74ff f5ce】 E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

以此类推，再下面的4字节则为IP目的地址，即“08 08 08 08”，转换成IP地址则为8.8.8.8，这是著名的Google-DNS服务器地址。

    0x0000: 4500 003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 【0808 0808】 bb44 0035 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

至此，网络层IP协议的首部20字节已经分析完了。再向下我们即将进入传输层UDP协议的包分析阶段。

相对于TCP包来说，UDP包的首部还是比较简单的，总共只有8个字节，如下绿色部分便是UDP首部部分：

    0x0000: 4500 003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 【bb44 0035 0027 b457】 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

UDP首部的前2个字节（如下棕色部分）为源端口，此处为“bb 44”，即47940；而接下来的2字节（如下粉色部分）为目的端口，值为“00 35”，即53（DNS服务）。

    0x0000: 4500 003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 【bb44】 【0035】 0027 b457 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

而接下来的2字节（如下棕色部分）则表示UDP包的总长度（报头+数据部分），此处的值为“00 27”，这算成十进制则为39，就表示此UDP包的总长度为39字节，减去首部的8字节外，还有31字节来存储真正要传输的数据。

    0x0000: 4500 003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 【0027】 【b457】 58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01             du.com.....

而UDP首部的最后两个字节（如上粉色部分）则是校验和部分，此处的值为0xb457。

再向下的部分（绿色部分），则是应用层协议的内容（本例中是DNS协议）了，如果你有兴趣，可以继续了解下DNS协议、HTTP协议、FTP协议等众多的应用层协议，这非常有利于大家在学习协议、追查网络问题时，透过现象看本质。

    0x0000: 4500 003b c341 0000 4011 3c93 74ff f5ce E..;.A..@.<.t...
    0x0010: 0808 0808 bb44 0035 0027 b457 【58f0 0100 .....D.5.'.WX...
    0x0020: 0001 0000 0000 0000 0377 7777 0562 6169 .........www.bai
    0x0030: 6475 0363 6f6d 0000 0100 01】             du.com.....


[1]http://www.cnblogs.com/ggjucheng/archive/2012/01/14/2322659.html
[2]http://roclinux.cn/?p=2820
[3]http://www.360doc.com/content/11/1013/12/1162697_155701557.shtml
