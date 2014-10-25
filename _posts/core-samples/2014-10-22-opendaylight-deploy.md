---
layout: post
category : sdn
tagline: "opendaylight 安装"
tags : [ opendaylight, sdn]
---
{% include JB/setup %}

本安装在 ubuntu-14.04-server 亲测

###预备条件

    sudo apt-get -y install openjdk-7-jdk
    sudo cat >> ~/.bashrc <<EOF
    export JAVA_HOME=/usr/lib/jvm/java-7-openjdk-amd64
    export JRE_HOME=${JAVA_HOME}/jre
    export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
    export PATH=${JAVA_HOME}/bin:${JRE_HOME}/bin:$PATH
    export MAVEN_OPTS='-Xmx1048m -XX:MaxPermSize=512m'
    EOF
    source ~/.bashrc

    sudo apt-get install -y maven
    mvn -v
    #Apache Maven 3.0.4
    #Maven home: /usr/share/maven
    #Java version: 1.7.0_65, vendor: Oracle Corporation
    #Java home: /usr/lib/jvm/java-7-openjdk-amd64/jre
    #Default locale: zh_CN, platform encoding: UTF-8
    #OS name: "linux", version: "3.8.0-29-generic", arch: "amd64", family: "unix"

    #如果需要安装mininet虚拟机,从这里下载： http://onlab.vicci.org/mininet-vm/mininet-2.1.0p2-140718-ubuntu-14.04-server-amd64-ovf.zip
    #minineet 教程： mininet.org/walkthrough/
    sudo apt-get install -y openvswitch-switch
    sudo apt-get install -y mininet
    sudo service openvswitch-controller stop
    sudo update-rc.d openvswitch-controller disable
    sudo mn --test pingall
    git clone git://github.com/mininet/mininet
    mininet/util/install.sh -fw


###Hydrogen 版本

####源码安装

    sudo apt-get -y install git
    mkdir ~/opendaylight
    cd ~/opendaylight
    git clone https://github.com/opendaylight/controller.git  ~/opendaylight/controller
    cd ~/opendaylight/controller
    git checkout -b hydrogen origin/stable/hydrogen
    cd ~/opendaylight/controller/opendaylight/distribution/opendaylight
    mvn clean install

    cd ~/opendaylight
    git clone https://github.com/opendaylight/openflowjava.git  ~/opendaylight/openflowjava
    cd ~/opendaylight/openflowjava
    git checkout -b hydrogen origin/stable/hydrogen

    cd ~/opendaylight
    git clone https://github.com/opendaylight/openflowplugin.git  ~/opendaylight/openflowplugin
    cd ~/opendaylight/openflowplugin
    git checkout -b hydrogen origin/stable/hydrogen
    cd ~/opendaylight/openflowplugin/distribution/base
    mvn clean install


####zip 安装

    wget -o
    unzip


###Helium 版本

####zip 安装

* 下载

    wget -o

* 解包

	$unzip distribution-karaf-0.2.0-Helium

* 切换到项目目录

	$cd distribution-karaf-0.2.0-Helium

* 修改配置

	$vi etc/org.apache.karaf.management.cfg

	将
	serviceUrl = service:jmx:rmi://0.0.0.0:${rmiServerPort}/jndi/rmi://0.0.0.0:${rmiRegistryPort}/karaf-${karaf.name}

	修改成

	serviceUrl = service:jmx:rmi://127.0.0.1:${rmiServerPort}/jndi/rmi://127.0.0.1:${rmiRegistryPort}/karaf-${karaf.name}，

* 开始安装

	$./bin/karaf

* 安装支持REST API的组件：

    opendaylight-user@root>feature:install odl-restconf

* 安装L2 switch和OpenFlow插件：

    opendaylight-user@root>feature:install odl-l2switch-switch
    opendaylight-user@root>feature:install odl-openflowplugin-all

* 安装基于karaf控制台的md-sal控制器功能，包括nodes、yang UI、Topology：

    opendaylight-user@root>feature:install odl-mdsal-apidocs ##此组件写错，很容易无法登录

* 安装DLUX功能

    opendaylight-user@root>feature:install odl-dlux-all

* 安装基于karaf控制台的ad-sal功能，包括Connection manager、Container、Network、Flows：

    opendaylight-user@root>feature:install odl-adsal-northbound

注意，上面一定要按照顺序安装组件。如果不能登陆，

	$ opendaylight-user@root>logout
	$ rm -rf ~/distribution-karaf-0.2.0-Helium/data
	$ ./bin/karaf clean

	重新重复上面安装组件过程

* 登陆

    在 firefox 或 chrome 登陆 http://IP:8181/dlux/index.html
    用户名：admin 密码：admin

* 验证

重新开一个窗口

	sudo mn --controller=remote,ip=IP,port=6633

重载登陆页面的 topology panle

[参考 SDNLab](http://www.sdnlab.com/?p=1931)
