---
layout: post
category : linux
tagline: "安装"
tags : [ operation, ftp]
---
{% include JB/setup %}


由于多次部署 ftp server,网上的资料太杂，故记录一个靠谱的。


###安装

测试环境

	Linux localhost 2.6.32-431.el6.x86_64 #1 SMP Fri Nov 22 03:15:09 UTC 2013 x86_64 x86_64 x86_64 GNU/Linux

第一步：安装vsftp pam db4

	yum install vsftpd pam* db4* -y

第二步：配置vsftpd服务的宿主

	useradd vsftpdadmin -s /sbin/nologin

第三步：建立ftp虚拟宿主帐户

	useradd ftpuser -s /sbin/nologin

第四步：配置vsftpd.conf

vim /etc/vsftpd/vsftpd.conf

修改

	anonymous_enable=YES  -->  anonymous_enable=NO   //不允许匿名用户访问，默认是允许。

	#chroot_list_enable=YES  -->  chroot_list_enable=YES      //不允许FTP用户离开自己主目录，默认是被注释掉的。
	#chroot_list_file=/etc/vsftpd/chroot_list --> chroot_list_file=/etc/vsftpd/chroot_list  //如果开启了chroot_list_enable=YES，那么一定要开启这个，这条是锁定登录用户只能家目录的位置，如果不开启用户登录时就会报500 OOPS的错。
	local_enable=YES      //允许本地用户访问，默认就是YES，不用改
	local_enable=YES      //允许本地用户访问，默认就是YES，不用改
	write_enable=YES      //允许写入，默认是YES，不用改
	local_umask=022       //上传后文件的权限掩码，不用改
	dirmessage_enable=YES //开启目录标语，默认是YES，开不开无所谓，我是默认就行
	xferlog_enable=YES    //开启日志，默认是YES，不用改
	connect_from_port_20=YES   //设定连接端口20
	xferlog_std_format=YES   //设定vsftpd的服务日志保存路径，不用改 
	#idle_session_timeout=600  -->  idle_session_timeout=600  //会话超时，客户端连接到ftp但未操作，默认被注释掉，可根据个人情况修改
	#async_abor_enable=YES  -->   async_abor_enable=YES     //支持异步传输功能，默认是注释掉的，去掉注释
	#ascii_upload_enable=YES  -->   ascii_upload_enable=YES   //支持ASCII模式的下载功能，默认是注释掉的，去掉注释
	#ascii_download_enable=YES  -->   ascii_download_enable=YES   //支持ASCII模式的上传功能，默认是注释掉的，去掉注释
	#ftpd_banner=Welcome to blah FTP service  //FTP的登录欢迎语，本身是被注释掉的，去不去都行
	#chroot_local_user=YES  --> chroot_local_user=YES    //禁止本地用户登出自己的FTP主目录，本身被注释掉，去掉注释
	pam_service_name=vsftpd  //设定pam服务下vsftpdd的验证配置文件名，不用改
	userlist_enable=YES    //拒绝登录用户名单，不用改
	TCP_wrappers=YES    //限制主机对VSFTP服务器的访问，不用改（通过/etc/hosts.deny和/etc/hosts.allow这两个文件来配置）

增加

	guest_enable=YES    //设定启用虚拟用户功能。
	guest_username=ftpuser   //指定虚拟用户的宿主用户。
	virtual_use_local_privs=YES   //设定虚拟用户的权限符合他们的宿主用户。
	user_config_dir=/etc/vsftpd/vconf   //设定虚拟用户个人Vsftp的配置文件存放路径

	注意：/etc/vsftp/chroot_list本身是不存在的，这要建立vim /etc/vsftp/chroot_list，然后将帐户输入一行一个，保存就可以了




第五步：建立日志文件

	touch /var/log/vsftpd.log    //日志文件
	chown vsftpdadmin.vsftpdadmin /var/log/vsftpd.log   //属于vsftpdadmin这个宿主

第六步：建立虚拟用户文件

	mkdir /etc/vsftpd/vconf/
	touch /etc/vsftpd/vconf/vir_user

第七步：建立虚拟用户

	vim /etc/vsftpd/vconf/vir_user

	virtualuser           //用户名
	12345678           //密码

	注意：第一行用户名，第二行是上一行用户名的密码，其他人的以此类推

第八步：生成数据库

	#db_load -T -t hash -f /etc/vsftpd/vconf/vir_user /etc/vsftpd/vconf/vir_user.db

第九步：设置数据库文件的访问权限

	#chmod 600 /etc/vsftpd/vconf/vir_user.db
	#chmod 600 /etc/vsftpd/vconf/vir_user

第十步：修改/etc/pam.d/vsftpd内容

	echo "auth required pam_userdb.so db=/etc/vsftpd/vconf/vir_user" >> /etc/pam.d/vsftpd
	echo "account required pam_userdb.so db=/etc/vsftpd/vconf/vir_user" >> /etc/pam.d/vsftpd 

第十一步：创建用户的配置文件

	注意：用户配置文件的名字要和创建的“虚拟用户”名字对应

	#touch /etc/vsftpd/vconf/virtualuser

	#vim /etc/vsftpd/vconf/virtualuser

	输入：
	local_root=/home/virtualuser           //虚拟用户的个人目录路径
	anonymous_enable=NO
	write_enable=YES
	local_umask=022
	anon_upload_enable=NO
	anon_mkdir_write_enable=NO
	idle_session_timeout=600
	data_connection_timeout=120
	max_clients=10
	max_per_ip=5
	local_max_rate=1048576     //本地用户的最大传输速度，单位是Byts/s，我设定的是10M

	以上根据自己的需要取舍

第十二步：建立虚拟用户目录

	如果不建立虚拟用户的个人目录，那么所有的虚拟用户登录后所在的目录都是同一个目录下
	# mkdir /home/virtualuser
	# chown ftpuser.ftpuser ./virtualuser
	# chmod 600 /home/virtualuser

配置就此完成，如果想增加新的用户，只要按照上面的第七步、第十步进行就可以了。


###问题

Q:450:读取目录列表失败

A:在vsftpd.conf加上了一句pasv_enable=NO

Q:外网无法登录，550错误，错误: 读取目录列表失败

A:路由器的配置，有开放TCP 20端口

Q:在链接ftp服务器连接失败,错误提示:  500 OOPS: cannot change directory:/home/XXX

A:
	setsebool -P ftpd_disable_trans1
	service vsftpd restart

	查看 chroot_list 文件中是否有你要增加的用户名

Q: ftp: connect: No route to host

A:
	vi /etc/sysconfig/iptables-config
	添加一行：IPTABLES_MODULES="ip_nat_ftp ip_conntrack_ftp"
	service iptables restart

	vi  /etc/sysconfig/iptables        # 打开配置文件 ，添加如下信息
	-A INPUT -m state --state NEW -m tcp -p tcp --dport 20 -j ACCEPT
 	-A INPUT -m state --state NEW -m tcp -p tcp --dport 21 -j ACCEPT

###其他

####创建用户

	sudo adduser --home=/home/test --disable-login  test
	sudo passwd test
	sudo chmod 555 -R  /home/ftp_server  #如果配置文件的wirte_enable为No，那么这里的用户目录不能有写权限

	sudo chmod 755 -R  /home/ftp_server  #如果配置文件的wirte_enable为YES，那么这里的用户目录不能有写权限
	service vsftpd restart


	注： ubuntu 系统千万不要用 useradd 命令，这是个大坑，我用了两个小时没有成功。


####设置

	如果设置为
	chroot_local_user＝YES
	chroot_list_enable=YES(这行可以没有, 也可以有)
	chroot_list_file=/etc/vsftpd.chroot_list
	那么, 凡是加在文件vsftpd.chroot_list中的用户都是不受限止的用户
	即, 可以浏览其主目录的上级目录.

	所以, 如果不希望某用户能够浏览其主目录上级目录中的内容,可以如上设置, 然后在文件vsftpd.chroot_list中不添加该用户即可(此时, 在该文件中的用户都是可以浏览其主目录之外的目录的).
	或者, 设置如下
	chroot_local_user＝NO
	chroot_list_enable=YES(这行必须要有, 否则文件vsftpd.chroot_list不会起作用)
	chroot_list_file=/etc/vsftpd.chroot_list
	然后把所有不希望有这种浏览其主目录之上的各目录权限的用户添加到文件vsftpd.chroot_list(此时, 在该文件中的用户都是不可以浏览其主目录之外的目录的)中即可(一行一个用户名).

