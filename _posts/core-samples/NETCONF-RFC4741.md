###Protocol Overview

The NETCONF protocol defines a simple mechanism through which a
network device can be managed, configuration data information can be
retrieved, and new configuration data can be uploaded and
manipulated.

NETCONF uses a simple RPC-based mechanism to facilitate communication
between a client and a server.

A key aspect of NETCONF is that it allows the functionality of the
management protocol to closely mirror the native functionality of the
device.

A NETCONF session is the logical connection between a network
administrator or network configuration application and a network
device. A device MUST support at least one NETCONF session and
SHOULD support multiple sessions.

                        Example
    +-------------+  +-----------------------------+
    |   Content   |  |     Configuration data      |          |
    +-------------+  +-----------------------------+
          |                       |
    +-------------+  +-----------------------------+
    | Operations  |  | <get-config>, <edit-config> |
    +-------------+  +-----------------------------+
          |                       |
    +-------------+  +-----------------------------+
    |    RPC      |  |     <rpc>, <rpc-reply>      |
    +-------------+  +-----------------------------+
          |                       |
    +-------------+  +-----------------------------+
    |  Transport  |  |                             |
    |  Protocol   |  |    BEEP, SSH, SSL, console  |
    +-------------+  +-----------------------------+


* 状态与配置分离

当配置和状态数据在一起的时候，会导致很多问题[第8页]。因此有专门针对
配置的 get-config 和 状态的 get 操作。

* RPC 通信模型
* 多协议支持
* 配置数据存储(Configuration Datastore)
    running
    candidate
    startup

##subtree filtering

###namespace

    <filter type="subtree">
        <top xmlns="http://example.com/schema/1.2/config"/>
    </filter>

只有出现 top 元素的节点，并且在 namespace http://example.com/schema/1.2/config
下的才匹配

###attribute

    <filter type="subtree">
        <t:top xmlns:t="http://example.com/schema/1.2/config">
            <t:interfaces>
                <t:interface t:ifName="eth0"/>
            </t:interfaces>
        </t:top>
    </filter>

只有在 http://example.com/schema/1.2/config 命名空间下的 interfaces 节点，并且
interface 中的 ifName 属性是 eth0 才匹配。

###Containment Node

上中的 top interfaces interface 都是 Containment Node

###Selection Node

    <filter type="subtree">
        <top xmlns="http://example.com/schema/1.2/config">
            <users/>
        </top>
    </filter>

这里 users 就是一个 Selection Node(即一个没有叶子节点的节点)

###Content Match Node

    <filter type="subtree">
        <top xmlns="http://example.com/schema/1.2/config">
            <users>
                <user>
                    <name>fred</name>
                </user>
            </users>
        </top>
    </filter>

name 是 Content Match Node

###例子

####No Filter

    <rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get/>
    </rpc>

    <rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
            <!-- ... entire set of data returned ... -->
        </data>
    </rpc-reply>

####Empty Filter

    <rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get>
            <filter type="subtree">
            </filter>
        </get>
    </rpc>

    <rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
        </data>
    </rpc-reply>


####Select the Entire <users> Subtree

    <rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get-config>
            <source>
                <running/>
            </source>
            <filter type="subtree">
                <top xmlns="http://example.com/schema/1.2/config">
                    <users/>
                </top>
            </filter>
        </get-config>
    </rpc>

OR

    <rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get-config>
            <source>
                <running/>
            </source>
            <filter type="subtree">
                <top xmlns="http://example.com/schema/1.2/config">
                    <users>
                        <user/>
                    </users>
                </top>
            </filter>
        </get-config>
    </rpc>

    <rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
            <top xmlns="http://example.com/schema/1.2/config">
                <users>
                    <user>
                        <name>root</name>
                        <type>superuser</type>
                        <full-name>Charlie Root</full-name>
                        <company-info>
                            <dept>1</dept>
                            <id>1</id>
                        </company-info>
                    </user>
                    <user>
                        <name>fred</name>
                        <type>admin</type>
                        <full-name>Fred Flintstone</full-name>
                        <company-info>
                            <dept>2</dept>
                            <id>2</id>
                        </company-info>
                    </user>
                    <user>
                        <name>barney</name>
                        <type>admin</type>
                        <full-name>Barney Rubble</full-name>
                        <company-info>
                            <dept>2</dept>
                            <id>3</id>
                        </company-info>
                    </user>
                </users>
            </top>
        </data>
    </rpc-reply>

####Select All <name> Elements within the <users> Subtree

    <rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get-config>
            <source>
                <running/>
            </source>
            <filter type="subtree">
                <top xmlns="http://example.com/schema/1.2/config">
                    <users>
                        <user>
                            <name/>
                        </user>
                    </users>
                </top>
            </filter>
        </get-config>
    </rpc>


    <rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
            <top xmlns="http://example.com/schema/1.2/config">
                <users>
                    <user>
                        <name>root</name>
                    </user>
                    <user>
                        <name>fred</name>
                    </user>
                    <user>
                        <name>barney</name>
                    </user>
                </users>
            </top>
        </data>
    </rpc-reply>

####One Specific <user> Entry


    <rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get-config>
            <source>
                <running/>
            </source>
            <filter type="subtree">
                <top xmlns="http://example.com/schema/1.2/config">
                    <users>
                        <user>
                            <name>fred</name>
                        </user>
                    </users>
                </top>
            </filter>
        </get-config>
    </rpc>

    <rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
            <top xmlns="http://example.com/schema/1.2/config">
                <users>
                    <user>
                        <name>fred</name>
                        <type>admin</type>
                        <full-name>Fred Flintstone</full-name>
                        <company-info>
                            <dept>2</dept>
                            <id>2</id>
                        </company-info>
                    </user>
                </users>
            </top>
        </data>
    </rpc-reply>

####Specific Elements from a Specific <user> Entry

    <rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get-config>
            <source>
                <running/>
            </source>
            <filter type="subtree">
                <top xmlns="http://example.com/schema/1.2/config">
                    <users>
                        <user>
                            <name>fred</name>
                            <type/>
                            <full-name/>
                        </user>
                    </users>
                </top>
            </filter>
        </get-config>
    </rpc>

    <rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
            <top xmlns="http://example.com/schema/1.2/config">
                <users>
                    <user>
                        <name>fred</name>
                        <type>admin</type>
                        <full-name>Fred Flintstone</full-name>
                    </user>
                </users>
            </top>
        </data>
    </rpc-reply>

####Multiple Subtrees

    <rpc message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get-config>
            <source>
                <running/>
            </source>
            <filter type="subtree">
                <top xmlns="http://example.com/schema/1.2/config">
                    <users>
                        <user>
                            <name>root</name>
                            <company-info/>
                        </user>
                        <user>
                            <name>fred</name>
                            <company-info>
                                <id/>
                            </company-info>
                        </user>
                        <user>
                            <name>barney</name>
                            <type>superuser</type>
                            <company-info>
                                <dept/>
                            </company-info>
                        </user>
                    </users>
                </top>
            </filter>
        </get-config>
    </rpc>

    <rpc-reply message-id="101" xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
            <top xmlns="http://example.com/schema/1.2/config">
                <users>
                    <user>
                        <name>root</name>
                        <company-info>
                            <dept>1</dept>
                            <id>1</id>
                        </company-info>
                    </user>
                    <user>
                        <name>fred</name>
                        <company-info>
                            <id>2</id>
                        </company-info>
                    </user>
                </users>
            </top>
        </data>
    </rpc-reply>

####Elements with Attribute Naming

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get>
            <filter type="subtree">
                <t:top xmlns:t="http://example.com/schema/1.2/stats">
                    <t:interfaces>
                        <t:interface t:ifName="eth0"/>
                    </t:interfaces>
                </t:top>
            </filter>
        </get>
    </rpc>

OR

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get>
            <filter type="subtree">
                <top xmlns:t="http://example.com/schema/1.2/stats">
                    <interfaces>
                        <ifName>eth0</ifName>
                    </interfaces>
                </top>
            </filter>
        </get>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <data>
            <t:top xmlns:t="http://example.com/schema/1.2/stats">
                <t:interfaces>
                    <t:interface t:ifName="eth0">
                        <t:ifInOctets>45621</t:ifInOctets>
                        <t:ifOutOctets>774344</t:ifOutOctets>
                    </t:interface>
                </t:interfaces>
            </t:top>
        </data>
    </rpc-reply>


###Protocol Operations

####element

* syntax
* encoding
* semantics

####operation

* get

<rpc message-id="101"
        xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <get>
        <filter type="subtree">
            <top xmlns="http://example.com/schema/1.2/stats">
                <interfaces>
                    <interface>
                        <ifName>eth0</ifName>
                    </interface>
                </interfaces>
            </top>
        </filter>
    </get>
</rpc>

<rpc-reply message-id="101"
        xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <data>
        <top xmlns="http://example.com/schema/1.2/stats">
            <interfaces>
                <interface>
                    <ifName>eth0</ifName>
                    <ifInOctets>45621</ifInOctets>
                    <ifOutOctets>774344</ifOutOctets>
                </interface>
            </interfaces>
        </top>
    </data>
</rpc-reply>

* get-config

例如
<rpc message-id="101"
        xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <get-config>
        <source>
            <running/>
        </source>
        <filter type="subtree">
            <top xmlns="http://example.com/schema/1.2/config">
                <users/>
            </top>
        </filter>
    </get-config>
</rpc>

<rpc-reply message-id="101"
        xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <data>
        <top xmlns="http://example.com/schema/1.2/config">
            <users>
                <user>
                    <name>root</name>
                    <type>superuser</type>
                    <full-name>Charlie Root</full-name>
                    <company-info>
                        <dept>1</dept>
                        <id>1</id>
                    </company-info>
                </user>
            <!-- additional <user> elements appear here... -->
            </users>
        </top>
    </data>
</rpc-reply>

**例如**

如果配置有多种格式可用，比如 XML text 格式，XML 命名空间可以指明具体
请求那种格式。

<rpc message-id="101"
        xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <get-config>
        <source>
            <running/>
        </source>
        <filter type="subtree">
            <!-- request a text version of the configuration -->
            <config-text xmlns="http://example.com/text/1.2/config"/>
        </filter>
    </get-config>
</rpc>

<rpc-reply message-id="101"
        xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <data>
        <config-text xmlns="http://example.com/text/1.2/config">
            <!-- configuration text... -->
        </config-text>
    </data>
</rpc-reply>


* edit-config

create merge delete replace

例如

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <edit-config>
            <target>
                <running/>
            </target>
            <config>
                <top xmlns="http://example.com/schema/1.2/config">
                    <interface>
                        <name>Ethernet0/0</name>
                        <mtu>1500</mtu>
                    </interface>
                </top>
            </config>
        </edit-config>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

例如

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <edit-config>
            <target>
                <running/>
            </target>
            <config>
                <top xmlns="http://example.com/schema/1.2/config">
                    <interface xc:operation="replace">
                        <name>Ethernet0/0</name>
                        <mtu>1500</mtu>
                        <address>
                            <name>192.0.2.4</name>
                            <prefix-length>24</prefix-length>
                        </address>
                    </interface>
                </top>
            </config>
        </edit-config>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

例如

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <edit-config>
            <target>
                <running/>
            </target>
            <config>
                <top xmlns="http://example.com/schema/1.2/config">
                    <interface xc:operation="delete">
                        <name>Ethernet0/0</name>
                    </interface>
                </top>
            </config>
        </edit-config>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

例如

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <edit-config>
            <target>
                <running/>
            </target>
            <default-operation>none</default-operation>
            <config xmlns:xc="urn:ietf:params:xml:ns:netconf:base:1.0">
                <top xmlns="http://example.com/schema/1.2/config">
                    <protocols>
                        <ospf>
                            <area>
                            <name>0.0.0.0</name>
                            <interfaces>
                                <interface xc:operation="delete">
                                    <name>192.0.2.4</name>
                                </interface>
                            </interfaces>
                            </area>
                        </ospf>
                    </protocols>
                </top>
            </config>
        </edit-config>
    </rpc>

    <rpc-reply message-id="101"
        <F2>xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
            <ok/>
    </rpc-reply>

* copy-config

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <copy-config>
            <target>
                <running/>
            </target>
            <source>
                <url>https://user@example.com:passphrase/cfg/new.txt</url>
            </source>
        </copy-config>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

* delete-config

    <rpc message-id="101"
    xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <delete-config>
            <target>
                <startup/>
            </target>
        </delete-config>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

* lock

一个客户端请求 lock 配置系统

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <lock>
            <target>
                <running/>
            </target>
        </lock>
    </rpc>

    <rpc-reply message-id="101"
        xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/> <!-- lock succeeded -->
    </rpc-reply>

另一个客户端试图 lock 一个已经 locked 的配置

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <lock>
            <target>
                <running/>
            </target>
        </lock>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <rpc-error> <!-- lock failed -->
            <error-type>protocol</error-type>
            <error-tag>lock-denied</error-tag>
            <error-severity>error</error-severity>
            <error-message>
                Lock failed, lock is already held
            </error-message>
            <error-info>
                <session-id>454</session-id>
                <!-- lock is held by NETCONF session 454 -->
            </error-info>
        </rpc-error>
    </rpc-reply>

* unlock

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <unlock>
            <target>
                <running/>
            </target>
        </unlock>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

* close-session

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <close-session/>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

* kill-session

<rpc message-id="101"
        xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
    <kill-session>
        <session-id>4</session-id>
    </kill-session>
</rpc>

##Capabilities

A NETCONF capability is a set of functionality that supplements the base NETCONF
specification. The capability is identified by a uniform resource identifier
(URI).

The device uses capabilities to announce the set of data models that the device
implements. The capability definition details the operation and constraints
imposed by data model.

###Capabilities Exchange

Capabilities are advertised in messages sent by each peer during session
establishment.

例子
    <hello xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <capabilities>
            <capability>
                urn:ietf:params:netconf:base:1.0
            </capability>
            <capability>
                urn:ietf:params:netconf:capability:startup:1.0
            </capability>
            <capability>
                http://example.net/router/2.3/myfeature
            </capability>
        </capabilities>
        <session-id>4</session-id>
    </hello>

###Writable-Running Capability

the device supports edit-config and copy-config operations where the <running>
configuration is the target.

target 为 running 的设备支持 edit-config 和 copy-config

####Identify

    urn:ietf:params:netconf:capability:writable-running:1.0

###Candidate Configuration Capability

可以理解 Candidate 是已经存储的配置数据，Writable-Running 的配置只有接受
Commit 请求后，才把之前处理的配置请求与 Candidate 配置同步。

####Identify
    urn:ietf:params:netconf:capability:candidate:1.0

####Operation

* commit
* get-config
* edit-config
* copy-config
* validate
* discard-changes
* lock
* unlock

###Confirmed Commit Capability

The :confirmed-commit capability indicates that the server will
support the <confirmed> and <confirm-timeout> parameters for the
<commit> protocol operation.

####Identify

    urn:ietf:params:netconf:capability:confirmed-commit:1.0

####Operation

* commit

例如

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <commit>
            <confirmed/>
            <confirm-timeout>120</confirm-timeout>
        </commit>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

###Rollback on Error Capability

####Identify

    urn:ietf:params:netconf:capability:rollback-on-error:1.0

####Operation

* edit-config

例如

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <edit-config>
            <target>
                <running/>
            </target>
            <error-option>rollback-on-error</error-option>
            <config>
                <top xmlns="http://example.com/schema/1.2/config">
                    <interface>
                        <name>Ethernet0/0</name>
                        <mtu>100000</mtu>
                    </interface>
                </top>
            </config>
        </edit-config>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

###Validate Capability

####Identify

    urn:ietf:params:netconf:capability:validate:1.0

####Operation

* validate

例如

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <validate>
            <source>
                <candidate/>
            </source>
        </validate>
    </rpc>

    <rpc-reply message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <ok/>
    </rpc-reply>

###Distinct Startup Capability

The device supports separate running and startup configuration
datastores.

####Identify

    urn:ietf:params:netconf:capability:startup:1.0

####Operation

* get-config    source
* copy-config   source target
* lock          target
* unlock        target
* validate      target

例如

    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <copy-config>
            <source>
                <running/>
            </source>
            <target>
                <startup/>
            </target>
        </copy-config>
    </rpc>

###URL Capability

The NETCONF peer has the ability to accept the <url> element in
<source> and <target> parameters

####Identify

    urn:ietf:params:netconf:capability:url:1.0?scheme={name,...}

####Operation

* edit-config
* copy-config   source target
* delete-config target
* validate      source

###XPath Capability

####Identify

    urn:ietf:params:netconf:capability:xpath:1.0

####Operation

* get-config
* get


    <rpc message-id="101"
            xmlns="urn:ietf:params:xml:ns:netconf:base:1.0">
        <get-config>
            <source>
                <running/>
            </source>
                <!-- get the user named fred -->
                <filter type="xpath" select="top/users/user[name=’fred’]"/>
        </get-config>
    </rpc>

###Topology

####IETF
####IANA
####URNs


