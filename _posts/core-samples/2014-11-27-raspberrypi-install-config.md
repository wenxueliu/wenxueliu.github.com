---
layout: post
category : smart hardware 
tagline: "树莓派安装与配置"
tags : [ raspberrypi ]
---
{% include JB/setup %}


##系统版本

    RASPBIAN
    Debian Wheezy
    Version:September 2014
    Release date:2014-09-09
    Default login:pi / raspberry
    URL:raspbian.org
    Kernel version:3.12
    Release notes:Link
    SHA-1:951a9092dd160ea06195963d1afb47220588ed84

##烧录系统到 SD 卡

unzip  2014-09-09-wheezy-raspbian.zip

运行 df -h 命令查看当前哪些设备已经挂载，插入SD卡后，再次运行 df -h，找出两次运行区别。区别就是多出这两个：

    /dev/sdb1
    /dev/sdb5

为了防止在写入镜像的时候有其他读取或写入，我们需要卸载设备。两个分区都要卸载

umount /dev/sdb1
umount /dev/sdb5

使用dd命令写入镜像至SD卡

    $ sudo dd bs=4M if=2014-01-07-wheezy-raspbian.img of=/dev/sdb

打开另一个命令行执行

    $ ps aux | grep dd  //找出 dd 的pid
    $sudo pkill -USR1 $pid; sleep 1; kill $pid  //pid 为 dd 的pid

这样可以查看进度

完成之后，取出 SD 卡，插入树莓派，给树莓派接上网线和 USB 电源。

##开机

然后，在路由器上查看到 树莓派 被 DHCP 服务器 分配到 192.168.1.101 这样的一个 IP地址。
那就先 SSH登录 吧。

    $ ssh -X  pi@192.168.1.101 //密码raspberry, X是必须的，如果要运行图形界面

    pi@raspberrypi ~ $ uname -a

        Linux raspberrypi 3.12.28+ #709 PREEMPT Mon Sep 8 15:28:00 BST 2014 armv6l GNU/Linux

##系统配置

    pi@raspberrypi ~ $ raspi-config

###扩展 SD 卡上可用的空间

选择

	1 Expand Filesystem              Ensures that all of the SD card storage is available to the OS

回车

###支持图形界面

选择

	3 Enable Boot to Desktop/Scratch Choose whether to boot into a desktop environment, Scratch, or the command-line

回车

选择

	Desktop Log in as user 'pi' at the graphical desktop 

回车

##加载网卡及配置网络

pi@raspberrypi ~ $ lsusb

	Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp. 
	Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
	Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp. 
	Bus 001 Device 004: ID 0bda:8176 Realtek Semiconductor Corp. RTL8188CUS 802.11n WLAN Adapter //网卡

pi@raspberrypi ~ $ lsmod

	Module                  Size  Used by
	snd_bcm2835            19584  0 
	snd_soc_bcm2708_i2s     6202  0 
	regmap_mmio             2818  1 snd_soc_bcm2708_i2s
	snd_soc_core          127841  1 snd_soc_bcm2708_i2s
	snd_compress            8259  1 snd_soc_core
	regmap_i2c              1661  1 snd_soc_core
	snd_pcm_dmaengine       5505  1 snd_soc_core
	regmap_spi              1913  1 snd_soc_core
	snd_pcm                83845  3 snd_bcm2835,snd_soc_core,snd_pcm_dmaengine
	snd_page_alloc          5132  1 snd_pcm
	snd_seq                55484  0 
	snd_seq_device          6469  1 snd_seq
	snd_timer              20998  2 snd_pcm,snd_seq
	leds_gpio               2079  0 
	led_class               4118  1 leds_gpio
	8192cu                550797  0 
	snd                    62252  7 snd_bcm2835,snd_soc_core,snd_timer,snd_pcm,snd_seq,snd_seq_device,snd_compress

###图形方式

    $wpa_gui

###命令行方式

    pi@raspberrypi ~ $ ps aux | grep wpa

	    pi        4162  0.0  0.1   3520   708 pts/0    S+   10:29   0:00 grep --color=auto wpa


    pi@raspberrypi ~ $ sudo cat /etc/wpa_supplicant/wpa_supplicant.conf 

        ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
        update_config=1

    pi@raspberrypi ~ $ sudo wpa_supplicant -B -Dwext -iwlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf 

        rfkill: Cannot open RFKILL control device
        ioctl[SIOCSIWAP]: Operation not permitted
        ioctl[SIOCSIWENCODEEXT]: Invalid argument
        ioctl[SIOCSIWENCODEEXT]: Invalid argument

    pi@raspberrypi ~ $ ps aux | grep wpa

        root      4180  0.0  0.2   5968   916 ?        Ss   10:30   0:00 wpa_supplicant -B -Dwext -iwlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf
        pi        4182  0.0  0.1   3520   708 pts/0    S+   10:30   0:00 grep --color=auto wpa


    pi@raspberrypi ~ $ wpa_cli

        wpa_cli v1.0
        Copyright (c) 2004-2012, Jouni Malinen <j@w1.fi> and contributors

        This program is free software. You can distribute it and/or modify it
        under the terms of the GNU General Public License version 2.

        Alternatively, this software may be distributed under the terms of the
        BSD license. See README and COPYING for more details.


        Could not connect to wpa_supplicant - re-trying

    如果出现上述信息表明权限问题

    pi@raspberrypi ~ $ sudo wpa_cli

        wpa_cli v1.0
        Copyright (c) 2004-2012, Jouni Malinen <j@w1.fi> and contributors
        This program is free software. You can distribute it and/or modify it
        under the terms of the GNU General Public License version 2.

        Alternatively, this software may be distributed under the terms of the
        BSD license. See README and COPYING for more details.


        Selected interface 'wlan0'

        Interactive mode

        > help
        commands:
        status [verbose] = get current WPA/EAPOL/EAP status
        ping = pings wpa_supplicant
        relog = re-open log-file (allow rolling logs)
        note <text> = add a note to wpa_supplicant debug log
        mib = get MIB variables (dot1x, dot11)
        help = show this usage help
        interface [ifname] = show interfaces/select interface
        level <debug level> = change debug level
        license = show full wpa_cli license
        quit = exit wpa_cli
        set = set variables (shows list of variables when run without arguments)
        get <name> = get information
        logon = IEEE 802.1X EAPOL state machine logon
        logoff = IEEE 802.1X EAPOL state machine logoff
        pmksa = show PMKSA cache
        reassociate = force reassociation
        preauthenticate <BSSID> = force preauthentication
        identity <network id> <identity> = configure identity for an SSID
        password <network id> <password> = configure password for an SSID
        new_password <network id> <password> = change password for an SSID
        pin <network id> <pin> = configure pin for an SSID
        otp <network id> <password> = configure one-time-password for an SSID
        passphrase <network id> <passphrase> = configure private key passphrase
            for an SSID
        bssid <network id> <BSSID> = set preferred BSSID for an SSID
        blacklist <BSSID> = add a BSSID to the blacklist
        blacklist clear = clear the blacklist
        blacklist = display the blacklist
        log_level <level> [<timestamp>] = update the log level/timestamp
        log_level = display the current log level and log options
        list_networks = list configured networks
        select_network <network id> = select a network (disable others)
        enable_network <network id> = enable a network
        disable_network <network id> = disable a network
        add_network = add a network
        remove_network <network id> = remove a network
        set_network <network id> <variable> <value> = set network variables (shows
            list of variables when run without arguments)
        get_network <network id> <variable> = get network variables
        save_config = save the current configuration
        disconnect = disconnect and wait for reassociate/reconnect command before
            connecting
        reconnect = like reassociate, but only takes effect if already disconnected
        scan = request new BSS scan
        scan_results = get latest scan results
        bss <<idx> | <bssid>> = get detailed scan result info
        get_capability <eap/pairwise/group/key_mgmt/proto/auth_alg> = get capabilies
        reconfigure = force wpa_supplicant to re-read its configuration file
        terminate = terminate wpa_supplicant
        interface_add <ifname> <confname> <driver> <ctrl_interface> <driver_param>
            <bridge_name> = adds new interface, all parameters but <ifname>
            are optional
        interface_remove <ifname> = removes the interface
        interface_list = list available interfaces
        ap_scan <value> = set ap_scan parameter
        scan_interval <value> = set scan_interval parameter (in seconds)
        bss_expire_age <value> = set BSS expiration age parameter
        bss_expire_count <value> = set BSS expiration scan count parameter
        stkstart <addr> = request STK negotiation with <addr>
        ft_ds <addr> = request over-the-DS FT with <addr>
        wps_pbc [BSSID] = start Wi-Fi Protected Setup: Push Button Configuration
        wps_pin <BSSID> [PIN] = start WPS PIN method (returns PIN, if not hardcoded)
        wps_check_pin <PIN> = verify PIN checksum
        wps_cancel Cancels the pending WPS operation
        wps_reg <BSSID> <AP PIN> = start WPS Registrar to configure an AP
        wps_ap_pin [params..] = enable/disable AP PIN
        wps_er_start [IP address] = start Wi-Fi Protected Setup External Registrar
        wps_er_stop = stop Wi-Fi Protected Setup External Registrar
        wps_er_pin <UUID> <PIN> = add an Enrollee PIN to External Registrar
        wps_er_pbc <UUID> = accept an Enrollee PBC using External Registrar
        wps_er_learn <UUID> <PIN> = learn AP configuration
        wps_er_set_config <UUID> <network id> = set AP configuration for enrolling
        wps_er_config <UUID> <PIN> <SSID> <auth> <encr> <key> = configure AP
        ibss_rsn <addr> = request RSN authentication with <addr> in IBSS
        suspend = notification of suspend/hibernate
        resume = notification of resume/thaw
        drop_sa = drop SA without deauth/disassoc (test command)
        roam <addr> = roam to the specified BSS
        fetch_anqp = fetch ANQP information for all APs
        stop_fetch_anqp = stop fetch_anqp operation
        interworking_select [auto] = perform Interworking network selection
        interworking_connect <BSSID> = connect using Interworking credentials
        anqp_get <addr> <info id>[,<info id>]... = request ANQP information
        sta_autoconnect <0/1> = disable/enable automatic reconnection
        tdls_discover <addr> = request TDLS discovery with <addr>
        tdls_setup <addr> = request TDLS setup with <addr>
        tdls_teardown <addr> = tear down TDLS with <addr>
        signal_poll = get signal parameters

        > scan
        OK
        > scan_results
        bssid / frequency / signal level / flags / ssid
        c4:a8:1d:e1:10:a6	2412	60	[WPA-PSK-CCMP][WPA2-PSK-CCMP][WPS][ESS]	D-Link_HRSJ
        14:75:90:3c:6c:b0	2437	47	[WPA-PSK-CCMP][WPA2-PSK-CCMP][ESS]	TP-LINK_ALEX
        9c:41:7c:7c:d2:a8	2412	100	[WPA-PSK-CCMP][ESS]	HAME_R1_cd2a
        ac:72:89:c5:09:be	2412	100	[WPA2-PSK-CCMP][ESS]	4#501
        d0:c7:c0:a7:e1:32	2412	48	[WPA-PSK-CCMP][WPA2-PSK-CCMP][WPS][ESS]	ZhangYiBin
        28:2c:b2:db:df:4e	2462	47	[WPA-PSK-CCMP][WPA2-PSK-CCMP][WPS][ESS]	ting269775
        50:9f:27:e2:01:64	2412	49	[WPA-PSK-CCMP][WPA2-PSK-CCMP][WPS][ESS]	ChinaNet-E93m
        c8:3a:35:32:a2:a0	2442	44	[WPA-PSK-CCMP][ESS]	Tenda_32A2A0
        14:e6:e4:43:8f:a0	2452	42	[WPA-PSK-CCMP][WPA2-PSK-CCMP][WPS][ESS]	TP-LINK_438FA0
        a8:15:4d:a2:a6:74	2437	29	[WPA-PSK-CCMP][WPA2-PSK-CCMP][WPS][ESS]	bee
        > add_network 
        0
        > set_network 0 ssid "4#501"
        OK
        > set_network 0 key_mgmt WPA2-PSK
        FAIL
        > set_network 0 key_mgmt WPA-PSK
        OK
        > set_network 0 psk "a1234567"
        OK
        > set_network 0 pairwise CCMP
        OK
        > set_network 0 group CCMP
        OK
        > set_network 0 proto RSN
        OK
        > enable_network 0
        OK
        > save_config 
        OK
        > quit

    如果命令行出现问题，最好通过图形界面方式重新配置

    pi@raspberrypi ~ $ sudo cat /etc/wpa_supplicant/wpa_supplicant.conf

        ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
        update_config=1

        network={
            ssid="4#501"
            psk="1234567"
            proto=RSN
            key_mgmt=WPA-PSK
            pairwise=CCMP
            auth_alg=OPEN
        }

    pi@raspberrypi ~ $ sudo ifdown wlan0 

        Internet Systems Consortium DHCP Client 4.2.2
        Copyright 2004-2011 Internet Systems Consortium.
        All rights reserved.
        For info, please visit https://www.isc.org/software/dhcp/

        Listening on LPF/wlan0/e8:4e:06:20:2e:78
        Sending on   LPF/wlan0/e8:4e:06:20:2e:78
        Sending on   Socket/fallback
        DHCPRELEASE on wlan0 to 192.168.253.1 port 67

    pi@raspberrypi ~ $ sudo ifup wlan0 

        wpa_supplicant: wpa-roam can only be used with the "manual" inet METHOD
        run-parts: /etc/network/if-pre-up.d/wpasupplicant exited with return code 1
        Internet Systems Consortium DHCP Client 4.2.2
        Copyright 2004-2011 Internet Systems Consortium.
        All rights reserved.
        For info, please visit https://www.isc.org/software/dhcp/

        Listening on LPF/wlan0/e8:4e:06:20:2e:78
        Sending on   LPF/wlan0/e8:4e:06:20:2e:78
        Sending on   Socket/fallback
        DHCPDISCOVER on wlan0 to 255.255.255.255 port 67 interval 8
        DHCPREQUEST on wlan0 to 255.255.255.255 port 67
        DHCPOFFER from 192.168.1.1
        DHCPACK from 192.168.1.1
        bound to 192.168.1.86 -- renewal in 3493 seconds.
        wpa_supplicant: wpa-roam can only be used with the "manual" inet METHOD
        run-parts: /etc/network/if-up.d/wpasupplicant exited with return code 1

    pi@raspberrypi ~ $ ifconfig wlan0 

        wlan0     Link encap:Ethernet  HWaddr e8:4e:06:20:2e:78
                inet addr:192.168.1.86  Bcast:192.168.1.255  Mask:255.255.255.0
                UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
                RX packets:29 errors:0 dropped:27 overruns:0 frame:0
                TX packets:6 errors:0 dropped:0 overruns:0 carrier:0
                collisions:0 txqueuelen:1000 
                RX bytes:4501 (4.3 KiB)  TX bytes:1316 (1.2 KiB)

###3G 上网（待验证）

    pi@raspberrypi ~ $ sudo apt-get install wvdial

    pi@raspberrypi ~ $ sudo wvdialconf

    会自动检测拨号设备

    在/etc下生成wvidial.conf配置文件

    配置文件略作修改即可使用

    sudo wvdial &

    可以看到拨号提示 获取ip和dns

    拨号完毕之后会回到命令行界面



##更新软件源

    pi@raspberrypi ~ $ sudo apt-get update


##安装远程桌面

###服务端

默认安装了 vnc-server，如果想用 rdesktop

    pi@raspberrypi ~ $ sudo apt-get install -y xrdp

###客户端

* 命令行

	pi@raspberrypi ~ $ rdesktop 192.168.1.84:3389   -u pi

* 图形软件

直接用 Remmina Remote Desktop Client


###配置摄像头

##UVL4

    pi@raspberrypi ~ $ wget http://www.linux-projects.org/listing/uv4l_repo/lrkey.asc && sudo apt-key
add ./lrkey.asc

    pi@raspberrypi ~ $ uv4l --port=9000  --user-password=user --enable-keepalive=yes 

    $ vim /etc/apt/sources.list 增加如下行

        deb http://www.linux-projects.org/listing/uv4l_repo/raspbian/ wheezy main

    $ sudo apt-get update
    $ sudo apt-get install uv4l uv4l-raspicam
    $ sudo apt-get install uv4l-raspicam-extras
    $ sudo service uv4l_raspicam restart
    $ sudo apt-get install uv4l-server

    $ uv4l --auto-video_nr --driver raspicam --encoding mjpeg --server-option '--port=9000' --server-option '--admin-password=password!'

这时在 firefox 中输入 http://raspberrypi:9000 可以看到有 config-panel 和 stream 可供选择，选择相应界面进入即可。

##问题

Q: /bin/bash: warning: setlocale: LC_ALL: cannot change locale (zh_CN.utf8)

A: sudo localedef -v -c -i zh_CN -f UTF-8 zh_CN.UTF-8

##参考

树莓派实验室
[UVL4 安装](http://www.linux-projects.org/modules/sections/index.php?op=viewarticle&artid=14)
[UVL4 案例](http://www.linux-projects.org/modules/sections/index.php?op=viewarticle&artid=16)

