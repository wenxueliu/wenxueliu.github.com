---
layout: post
category : linux
comments : true
tags : [network, tools, tutorial]

---
{% include JB/setup %}

整理by文刀旋子

#前言

虽然网络编程的socket大家很多都会操作，但是很多还是不熟悉socket编程中，底层TCP/IP协议的交互过程，本文会一个简单的客户端程序和服务端程序的交互过程，使用tcpdump抓包，实例讲解客户端和服务端的TCP/IP交互细节。

###TCP/IP协议

IP头和TCP头格式如下:

Internet Header Format

        0                   1                   2                   3           
        0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |Version|  IHL  |Type of Service|          Total Length         |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |         Identification        |Flags|      Fragment Offset    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |  Time to Live |    Protocol   |         Header Checksum       |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                       Source Address                          |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                    Destination Address                        |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
       |                    Options                    |    Padding    |
       +-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+-+
   
TCP Header Format

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

单单看这些头会比较枯燥，后面会根据一个简单的客户端和服务端的TCP/IP报文交互实例讲解这些报文头的格式和含义

##简单的客户端和服务端

客户端和服务端是我们写的一个简单客户端程序，运行在Linux上。

客户端和服务端的功能如下:

    1.客户端从标准输入读入一行，发送到服务端
    2.服务端从网络读取一行，然后输出到客户端
    3.客户端收到服务端的响应，输出这一行到标准输出

###服务端代码如下:

    include  <unistd.h>
    include  <sys/types.h>       /* basic system data types */
    include  <sys/socket.h>      /* basic socket definitions */
    include  <netinet/in.h>      /* sockaddr_in{} and other Internet defns */
    include  <arpa/inet.h>       /* inet(3) functions */

    include <stdlib.h>
    include <errno.h>
    include <stdio.h>
    include <string.h>

    define MAXLINE 1024
    //typedef struct sockaddr  SA;
    void handle(int connfd);

    int  main(int argc, char **argv)
    {
        int  listenfd, connfd;
        int  serverPort = 6888;
        int  listenq = 1024;
        pid_t   childpid;
        char buf[MAXLINE];
        socklen_t socklen;

        struct sockaddr_in cliaddr, servaddr;
        socklen = sizeof(cliaddr);

        bzero(&servaddr, sizeof(servaddr));
        servaddr.sin_family = AF_INET;
        servaddr.sin_addr.s_addr = htonl(INADDR_ANY);
        servaddr.sin_port = htons(serverPort);

        listenfd = socket(AF_INET, SOCK_STREAM, 0);
        if (listenfd < 0) {
            perror("socket error");
            return -1;
        }
        if (bind(listenfd, (struct sockaddr *) &servaddr, socklen) < 0) {
            perror("bind error");
            return -1;
        }
        if (listen(listenfd, listenq) < 0) {
            perror("listen error");    
            return -1;
        }
        printf("echo server startup,listen on port:%d\n", serverPort);
        for ( ; ; )  {
            connfd = accept(listenfd, (struct sockaddr *)&cliaddr, &socklen);
            if (connfd < 0) {
                perror("accept error");
                continue;
            }

            sprintf(buf, "accept form %s:%d\n", inet_ntoa(cliaddr.sin_addr), cliaddr.sin_port);
            printf(buf,"");
            childpid = fork();
            if (childpid == 0) { /* child process */
                close(listenfd);    /* close listening socket */
                handle(connfd);   /* process the request */
                exit (0);
            } else if (childpid > 0)  {
                close(connfd);          /* parent closes connected socket */
            } else {
                perror("fork error");
            }
        }
    }


    void handle(int connfd)
    {
        size_t n;
        char  buf[MAXLINE];

        for(;;) {
            n = read(connfd, buf, MAXLINE);
            if (n < 0) {
                if(errno != EINTR) {
                    perror("read error");
                    break;
                }
            }
            if (n == 0) {
                //connfd is closed by client
                close(connfd);
                printf("client exit\n");
                break;
            }
            //client exit
            if (strncmp("exit", buf, 4) == 0) {
                close(connfd);
                printf("client exit\n");
                break;
            }
            write(connfd, buf, n); //write maybe fail,here don't process failed error
        } 
    } 

###客户端代码如下:

    include  <unistd.h>
    include  <sys/types.h>       /* basic system data types */
    include  <sys/socket.h>      /* basic socket definitions */
    include  <netinet/in.h>      /* sockaddr_in{} and other Internet defns */
    include  <arpa/inet.h>       /* inet(3) functions */
    include <netdb.h> /*gethostbyname function */

    include <stdlib.h>
    include <errno.h>
    include <stdio.h>
    include <string.h>

    define MAXLINE 1024

    void handle(int connfd);

    int main(int argc, char **argv)
    {
        char *servInetAddr = "127.0.0.1";
        int servPort = 6888;
        char buf[MAXLINE];
        int connfd;
        struct sockaddr_in servaddr;

        if (argc == 2) {
            servInetAddr = argv[1];
        }
        if (argc == 3) {
            servInetAddr = argv[1];
            servPort = atoi(argv[2]);
        }
        if (argc > 3) {
            printf("usage: echoclient <IPaddress> <Port>\n");
            return -1;
        }

        connfd = socket(AF_INET, SOCK_STREAM, 0);

        bzero(&servaddr, sizeof(servaddr));
        servaddr.sin_family = AF_INET;
        servaddr.sin_port = htons(servPort);
        inet_pton(AF_INET, servInetAddr, &servaddr.sin_addr);

        if (connect(connfd, (struct sockaddr *) &servaddr, sizeof(servaddr)) < 0) {
            perror("connect error");
            return -1;
        }
        printf("welcome to echoclient\n");
        handle(connfd);     /* do it all */
        close(connfd);
        printf("exit\n");
        exit(0);
    }

    void handle(int sockfd)
    {
        char sendline[MAXLINE], recvline[MAXLINE];
        int n;
        for (;;) {
            if (fgets(sendline, MAXLINE, stdin) == NULL) {
                break;//read eof
            }
            /*
            //也可以不用标准库的缓冲流,直接使用系统函数无缓存操作
            if (read(STDIN_FILENO, sendline, MAXLINE) == 0) {
                break;//read eof
            }
            */

            n = write(sockfd, sendline, strlen(sendline));
            n = read(sockfd, recvline, MAXLINE);
            if (n == 0) {
                printf("echoclient: server terminated prematurely\n");
                break;
            }
            write(STDOUT_FILENO, recvline, n);
            //如果用标准库的缓存流输出有时会出现问题
            //fputs(recvline, stdout);
        }
    }

编译服务端代码: `gcc echoserver.c -o echoserver`

编译客户端代码: `gcc echoclient.c -o echoclient`

###TCP/IP连接建立，交互，关闭

####首先我们要启用tcpdump监控客户端和服务端的报文:

    tcpdump -S -nn -vvv -i lo port 6888

`-S` : 打印TCP 数据包的顺序号时, 使用绝对的顺序号, 而不是相对的顺序号

`-nn` : 表示不进行端口到名称的转换

`-vvv` : 表示产生尽可能详细的协议输出

`-i` : lo表示只监控网卡lo设备，默认是监控第一个网络设备。

`port` : 6888表示只监控端口6888的相关监控数据，包括从6888端口接收和从6888端口发送的报文。


###接着我们要启动服务端:

    ./echoserver

###再启动客户端:

    ./echoclient

###建立连接

客户端程序一启动，就会connect服务端，tcpdump对应的输出如下:

    13:27:45.927137 IP (tos 0x0, ttl  64, id 304, offset 0, flags [DF], proto: TCP (6), length: 60) 127.0.0.1.60534 > 127.0.0.1.6888: S, cksum 0x5f32 (correct), 2584692379:2584692379(0) win 32792 <mss 16396,sackOK,timestamp 10962859 0,nop,wscale 6>
    13:27:45.927254 IP (tos 0x0, ttl  64, id 0, offset 0, flags [DF], proto: TCP (6), length: 60) 127.0.0.1.6888 > 127.0.0.1.60534: S, cksum 0x3648 (correct), 2589673026:2589673026(0) ack 2584692380 win 32768 <mss 16396,sackOK,timestamp 10962860 10962859,nop,wscale 6>
    13:27:45.927265 IP (tos 0x0, ttl  64, id 305, offset 0, flags [DF], proto: TCP (6), length: 52) 127.0.0.1.60534 > 127.0.0.1.6888: ., cksum 0x1d6a (correct), 2584692380:2584692380(0) ack 2589673027 win 513 <nop,nop,timestamp 10962860 10962860>

这里是TCP连接的三握手的报文交互，其中协议各个字段的含义如下:

`tos` : 表示服务类型，4bit的tos分别表示最小时延，最大吞吐量，最高可靠性，最小费用。这里都是0，表示一般服务，其余4bit废用，置0.

`TTL(time-to-live)` : 生存时间字段设置了数据报可以经过的最多路由器数。它指定了数据报的生存时间。TTL的初始值由源主机设置（通常为32或64），一旦经过一个处理它的路由器，它的值就减去1。当该字段的值为0时，数据报就被丢弃，并发送ICMP报文通知源主机。

`id` : 对应IP报文头的Identification,用于IP分片重组。

`offset` : 也用于IP分片重组，表示相对于原始未分片的报文的位置。

`flags` : MF表示有更多分片，DF表示不分片，这里是DF，未使用分片，所以id和offset的值都可以忽略。

`proto` : 表示协议，可以是TCP，UDP等，这里是TCP。

`length` : 总长度字段，是指整个IP数据报的长度(至于首部长度这里没有给出，首部长度给出首部中32 bit字的数目。需要这个值是因为任选字段的长度是可变的。这个字段占4 bit，因此TCP最多有60字节的首部。然而没有任选字段，正常的长度是20字节)。

`127.0.0.1.60534 > 127.0.0.1.6888` : 表示数据是从IP为127.0.0.1端口为60534发送到IP为127.0.0.1端口为6888。分别对应的IP报文头的源地址和目的地址，以及TCP报文头的源端口和目的端口。

`S` : 当建立一个新的连接时，SYN标志变1。序号字段包含由这个主机选择的该连接的初始序号ISN（Initial Sequence Number）。该主机要发送数据的第一个字节序号为这个ISN加1，因为SYN标志消耗了一个序号,这里客户端的ISN是2584692379,服务端 的ISN是2589673026。

`chksum` : 16位检验和，这里有IP首部检验和和TCP报文段(包括TCP首部和数据)检验和，具体是哪个检验和不详。

`2584692379:2584692379(0)` : 表示,第一个2584692379表示TCP报文段的序列号，(0)表示数据长度是0，即没有数据，第二个2584692379是第一个2584692379+数据长度计算出来的。TCP是可靠连接，三握手的最大目的是为了初始化双方的ISN。假设客户端连接服务端，发送数据，刚好网络比较慢，在传输过程中，客户端和服务端已经都重启了，重新建立连接发送数据，发送过程中，服务端收到已经之前客户端的数据，发现ISN非法，就会抛弃这个包，不会对现有的服务造成影响。 这个只是ISN的一方面的作用。

`win` : TCP窗口大小，通知对方，发送方最多还可以接收的数据量，用于TCP的拥塞控制。第一个报文表示客户端通知服务端，客户端可以接受的数据的缓存区最大是 32792个字节。服务端通知客户端，服务端最多可以接受的缓存区最大是32768，这个窗口大小在一方接受数据，却没有read的时候，窗口会逐渐减小，直至为0，最后对方不可以发送任何数据(如果要做该测试，需要发送的数据量大概接近65535，因为窗口的缓存区也会在剩余容量减小时，自动增加总共容量，直到总共容量接近65535，接下来就会看到win越来越小，直至0)。

`ack` : TCP是可靠连接，所以收到发送方的数据，接受方就会发送ack确认，告诉发送方，接受方已经接收到数据，否则，发送方认为数据没有发送成功，重复发送数 据。第二个包有ack 2584692380，其中2584692380是第一个报文包的2584692379:2584692379(0)的第二个2584692379+1的 值。

`mss 16396,sackOK,timestamp 10962859 0,nop,wscale 6 ` : 这里表示IP报文头的可选字段，mss是最小最大分段大小,这里是16396，表示一个TCP报文段发送的数据最大可以是16396个字节,可能是lo设备的关系，这个mss很大，一般都是**mss = MTU 1500 个字节 - IP数据报文头20个字节 - TCP报文头20个字节 = 1460个字节**。wscale是TCP窗口扩大选项的窗口扩大因子，用于扩大TCP通告窗口，使TCP的窗口定义从16bit增加为32bit。这里的 wscale是6，那么实际窗口是513左移6位，既513 X 64 = 32832，这个选项只在一个SYN报文中有意义。其他选项不详，具体参考RFC。

###交互

建立连接之后，我们通过客户端分别发送a和123到服务端，客户端后台显示如下:

[root@localhost simpletcpip]# ./echoclient 

    welcome to echoclient
    a
    a
    123
    123

tcpdump对应的输出是:
    
    13:27:48.248592 IP (tos 0x0, ttl  64, id 306, offset 0, flags [DF], proto: TCP (6), length: 54) 127.0.0.1.60534 > 127.0.0.1.6888: P, cksum 0xfe2a (incorrect (-> 0xb344), 2584692380:2584692382(2) ack 2589673027 win 513 <nop,nop,timestamp 10965181 10962860>
    13:27:48.248739 IP (tos 0x0, ttl  64, id 495, offset 0, flags [DF], proto: TCP (6), length: 52) 127.0.0.1.6888 > 127.0.0.1.60534: ., cksum 0x0b47 (correct), 2589673027:2589673027(0) ack 2584692382 win 512 <nop,nop,timestamp 10965181 10965181>
    13:27:48.249061 IP (tos 0x0, ttl  64, id 496, offset 0, flags [DF], proto: TCP (6), length: 54) 127.0.0.1.6888 > 127.0.0.1.60534: P, cksum 0xfe2a (incorrect (-> 0xaa32), 2589673027:2589673029(2) ack 2584692382 win 512 <nop,nop,timestamp 10965181 10965181>
    13:27:48.249085 IP (tos 0x0, ttl  64, id 307, offset 0, flags [DF], proto: TCP (6), length: 52) 127.0.0.1.60534 > 127.0.0.1.6888: ., cksum 0x0b43 (correct), 2584692382:2584692382(0) ack 2589673029 win 513 <nop,nop,timestamp 10965182 10965181>
    13:27:49.544830 IP (tos 0x0, ttl  64, id 308, offset 0, flags [DF], proto: TCP (6), length: 56) 127.0.0.1.60534 > 127.0.0.1.6888: P, cksum 0xfe2c (incorrect (-> 0xa1eb), 2584692382:2584692386(4) ack 2589673029 win 513 <nop,nop,timestamp 10966477 10965181>
    13:27:49.544987 IP (tos 0x0, ttl  64, id 497, offset 0, flags [DF], proto: TCP (6), length: 56) 127.0.0.1.6888 > 127.0.0.1.60534: P, cksum 0xfe2c (incorrect (-> 0x9cd8), 2589673029:2589673033(4) ack 2584692386 win 512 <nop,nop,timestamp 10966477 10966477>
    13:27:49.545010 IP (tos 0x0, ttl  64, id 309, offset 0, flags [DF], proto: TCP (6), length: 52) 127.0.0.1.60534 > 127.0.0.1.6888: ., cksum 0x011c (correct), 2584692386:2584692386(0) ack 2589673033 win 513 <nop,nop,timestamp 10966477 10966477>

第一个报文是客户端给服务端发送了一个a数据，P表示push标志，发送方使用该标志通知接收方将所收到的数据全部提交给接收进程。这里的数据包括与PUSH一起传送的数据以及接收方TCP已经为接收进程收到的其他数据。长度为54是因为a数据后面还有一个\n(输入a然后回车导致的),所以是2个字节，比正常情况下没有数据的52个字节多了2个字节。

第二个报文是服务端接受了a数据后，发送给客户端的ack确认。

第三个报文和第四个报文分别是服务端发给客户端的回显a，以及客户端的ack确认。

第五个报文是客户端给服务端发送了123数据。

第六个报文，是服务端给客户端发送回显123，同时合并了对客户端的ack确认。

第七个报文是客户端对服务端的回显123数据的ack确认。

###关闭连接

直接在客户端的控制终端执行Ctrl+C结束客户端程序，就可以关闭TCP连接，tcpdump输出如下:

    13:38:10.081895 IP (tos 0x0, ttl  64, id 310, offset 0, flags [DF], proto: TCP (6), length: 52) 127.0.0.1.60534 > 127.0.0.1.6888: F, cksum 0x897d (correct), 2584692386:2584692386(0) ack 2589673033 win 513 <nop,nop,timestamp 11586913 10966477>
    13:38:10.081987 IP (tos 0x0, ttl  64, id 498, offset 0, flags [DF], proto: TCP (6), length: 52) 127.0.0.1.6888 > 127.0.0.1.60534: F, cksum 0x11e0 (correct), 2589673033:2589673033(0) ack 2584692387 win 512 <nop,nop,timestamp 11586913 11586913>
    13:38:10.081993 IP (tos 0x0, ttl  64, id 311, offset 0, flags [DF], proto: TCP (6), length: 52) 127.0.0.1.60534 > 127.0.0.1.6888: ., cksum 0x11df (correct), 2584692387:2584692387(0) ack 2589673034 win 513 <nop,nop,timestamp 11586913 11586913>

关闭的请求由客户端发出，第一个报文是客户端发给服务端，F表示Fin，发送关闭连接请求。这个关闭连接请求是由于客户端退出，操作系统回收客户端资源，自动发出的。当然，如果我们在客户端输入Ctrl+D,最后执行close(connfd)，关闭连接请求的报文就会发送。

第二个报文是服务端发给客户端的关闭连接请求，当客户端关闭连接是，服务端处理客户端请求的handle函数中

    if (n == 0) {
                //connfd is closed by client
                close(connfd);
                printf("client exit\n");
                break;
    }

执行close(connfd),对应的第二个报文就会发送。
第三个报文只是客户端对服务端的关闭连接请求报文的确认。

##总结

本文基于tcpdump和一个简单的客户端和服务端，实例讲解了TCP/IP协议，不止有协议的含义，而且有TCP/IP连接的建立，交互，关闭的 一些细节。由于篇幅和tcpdump的输出问题，未能将IP协议和TCP协议的报文头的每个字段含义都讲解一次，如果大家希望可以进一步了解，可以参考 RFC 791 - Internet Protocol和RFC 793 - Transmission Control Protocol，还有TCP/IP协议卷一的相关内容。
