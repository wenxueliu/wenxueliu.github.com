
### Opendaylight Architectural Principles

https://wiki.opendaylight.org/view/OpenDaylight_Controller:Architectural_Principles

* Runtime Modularity and Extensibility

Allow for a modular, extensible controller that supports installation, removal and updates of service implementations within a running controller, also known as in service software upgrade (ISSU) 

* Multiprotocol Southbound

Allow for more than one protocol interface with network elements with diverse capabilities southbound from the controller. 

* Service Abstraction Layer (SAL)

Where possible, allow for multiple southbound protocols to present the same northbound service interfaces. 

* Support for Multitenancy/Slicing

Allow for the network to be logically (and/or physically) split into different slices or tenants with parts of the controller, modules, explicitly dedicated to one or a subset of these slices. This includes allowing the controller to present different views of the controller depending on which slice the caller is from. 

* Consistent Clustering

Clustering that gives fine-grained redundancy and scale out while insuring network consistency. 



###Controller

		Clustering Manager 			Manages shared cache across controller instances
		Container Manager 			Manages Network Slicing
		Switch Manager 				Handles SB devices Information
		Statistics Manager 			Collects Statistics information
		Topology Manager 			Builds network topology
		Host Tracker 				Tracks about connected hosts
		Forwarding Rules Manager 	Installs Flows on SB devices
		ARP Handler 				Handles ARP messages
		Forwarding Manager 			Installs Routes and tracks next-hop
		OpenFlow Plugin 			Interacts with OF switches
		Netconf Plugin 				Interacts with Netconf switches 

###YANG

[RFC6020]

YANG is a modeling language written to support netconf based devices. But in
opendaylight we are using it to describe the structure of data provided by
controller components.

YANG is basically a schema that defines how data will be stored in MD-SAL data
store and what operations can be performed on this data e.g.
Create/Retrieve/Update/Delete (CRUD).

In opendaylight, yang is being used as a general purpose modeling language.
There are two code generators that drive this:

####MD-SAL Generator

It uses **Restconf** i.e provides a restconf endpoint. MD-SAL generator is used
as typesafe layer to ease development of datastore related components so that
when component A wants to communicate with B over md-sal (which can be local or
remote call), it does not have to care about (de)serialization. MD-SAL as whole
is driven by yang models, basically using it as validation, similar to xsd.


#####Config Generator

It uses **Netconf** i.e. provides a netconf endpoint. Config subsystem also uses
models to generate code. It’s scope is to provide dependency injection and
configuration during server runtime. This is somewhat similar to blueprint in
OSGi, but externalizes the XML from jar files. So you can connect to running ODL
via netconf and push xml that will reconfigure the server. Let’s say you want to
mount new netconf/bgp/openflow node, you just connect via netconf and push the
change. Yang in config subsystem is used mainly to generate config java skeletons,
and also as type safe layer, so that netconf client can do partial validation.

Both restconf and netconf are just protocols to get/edit configuration data,
operational data, call rpcs and receive notifications. A good analogy for me of
md-sal is XML database. Config is more like spring with dynamic reconfiguration.
But both can be looked as xml databases. So MD-SAL database is accessible
through Restconf and Config subsystem database is accessible through Netconf.

Yang models are used in the MD-SAL and in MD-SAL-based applications to define
all APIs: inter-component APIs, plugin APIs, northbound APIs, etc. Yang models
are used to generate Java APIs at compile time with OpenDaylight Yang Tools and
to render REST APIs at run time according to the RESTCONF specification. Plugins
designed for MD-SAL define yang models for their northbound REST APIs, which are
then exposed to applications via an MD-SAL RESTCONF adapter.”

###NETCONF

IETF configuration management protocol [RFC 6241](https://tools.ietf.org/html/rfc6241)

The NETCONF protocol defines a simple mechanism through which a network device
can be managed, configuration data information can be retrieved, and new
configuration data can be uploaded and manipulated.

A protocol that defines configuration datastores and a set of Create, Retrieve,
Update, Delete (CRUD) operations that can be used to access these datastores.
There are three NETCONF datastores – candidate, running & startup.

NETCONF uses a simple RPC-based mechanism to facilitate communication between a
client and a server.  The client can be a script or application typically
running as part of a network manager.  The server is typically a network device.

The NETCONF protocol uses a remote procedure call (RPC) paradigm.  A client
encodes an RPC in XML and sends it to a server using a secure,
connection-oriented session.  The server responds with a reply encoded in XML.

The <rpc> element is used to enclose a NETCONF request sent from the client to
the server. The <rpc-reply> message is sent in response to an <rpc> message.

NETCONF allows a client to discover the set of protocol extensions supported by
a server.  These “capabilities” permit the client to adjust its behavior to take
advantage of the features exposed by the device.

###Datastore

Datastore: A conceptual place to store and access information. A datastore might
be implemented, for example, using files, a database, flash memory locations, or
combinations thereof.

Configuration datastore: The datastore holding the complete set of configuration
data that is required to get a device from its initial default state into a
desired operational state.

State data: The additional data on a system that is not configuration data such
as read-only status information and collected statistics.  Candidate configuration
datastore: A configuration datastore that can be manipulated without impacting
the device’s current configuration and that can be committed to the running
configuration datastore. Not all devices support a candidate configuration datastore.

Running configuration datastore: A configuration datastore holding the complete
configuration currently active on the device. The running configuration datastore
always exists.

Startup configuration datastore: The configuration datastore holding the
configuration loaded by the device when it boots. Only present on devices that
separate the startup configuration datastore from the running configuration
datastore.


###RESTConf

[RESTConf 协议](https://datatracker.ietf.org/doc/draft-bierman-netconf-restconf/)
[Rest API(截止日期20141030)](https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Restconf_API_Explorer)

A REST like protocol running over HTTP for accessing data defined in YANG using
datastores defined in NETCONF.

RESTCONF is an IETF draft that describes how to map a YANG specification to a
RESTful interface.

The REST-like API is not intended to replace NETCONF, but rather provide an
additional simplified interface that follows REST-like principles and is
compatible with a resource-oriented device abstraction.

RESTCONF allows access to datastores locating in controller. There are two
datastores:Each request must start with URI /restconf

* Config – contains data inserted via controller
* Operational – contains data inserted via network

RESTCONF listens on port 8080 for HTTP requests

RESTCONF supports OPTIONS, GET, PUT, POST, DELETE operations.

Request and response data can be in XML or JSON format. XML has structure
according to yang by XML-YANG and JSON by JSON-YANG.

不适合基于驱动（event-based）的交互

In OpenFlow, there is one important event that signals to the control
application that a packet without matching flow table entry has arrived  at a
switch (packet-in event). In this case, a controll application implementing the
control logic has to decide what to do: dropping the packet, forwarding it, or
setting up a flow table entry for similar packets. Because of this limitation,
the REST interface is limited to proactive flow programming, where the control
application proactively programs the flow table of the switches; reactive flow
programming where the control application reacts on packet-in events is
implemented in OpenDaylight using OSGI components.

The basic job of the Flow Programmer is to query and change the state of these
resources by returning, adding, or deleting flows. The state of a resource can
be represented in different formats, namely, XML or JSON.

[例子](http://www.frank-durr.de/?p=68)

###MD-SAL

<img src="{{ IMAGE_PATH }}/opendaylight/MD_SAL.png" alt="MD_SAL"
title="opendaylight MD_SAL" />

Flexibility, but common framework and programming model 

    Support API governance

Run-time Extensibility:	

  Augment existing functionality
  Load new models (extends controller's	functionality)	

Dynamic late binding

Runtime & Compile time code generation

Avoid module/sub-­‐system hotspots	

Performance & scale

必读

* [YANG 创建 project Guide](https://wiki.opendaylight.org/view/Maven_Archetypes:odl-model-project)
* YANG Tool 使用参照[这里](https://wiki.opendaylight.org/view/YANG_Tools:Maven_Plugin_Guide) 
* [Code-Gernerate](https://wiki.opendaylight.org/view/Yang_Tools:YANG_to_Java_Mapping)
* [Code-Gernerate1](https://wiki.opendaylight.org/view/Yang_Tools:Code_Generation_Demo:YANG2JAVA_Mapping_%28Flow_example%29)
	controller/opendaylight/md-sal/samples/toaster
	controller/opendaylight/md-sal/samples/toaster-consumer
	controller/opendaylight/md-sal/samples/toaster-provider

参考

* [全部 API](https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Model_Reference)
* [YANG Modules](https://wiki.opendaylight.org/view/YANG_Tools:Available_Models)

####operational store  vs config store

The config store is where “requests” are stored and the operational store is
where “network state as discovered from the network” is stored. So Flows are
requested by being placed in the config store, but after they are configured on
the NE and ODL “discovers” them that data is put in the operational store.

###AD-SAL VS MD-SAL

<img src="{{ IMAGE_PATH }}/opendaylight/AD_SAL_vs_MD_SAL.png" alt="SAL"
title="opendaylight AD-SAL VS MD_SAL" />

AD-SAL is statically defined at compile/build time, providers and consumers of
events/data are  essentially hard wired directly, any REST interface was
manually defined, and service interfaces / data had to be manually written.

With MD-SAL this is essentially driven by the YANG model either through code
generation at compile time and loaded into the controller when the plugin OSGI
bundle is loaded into the controller or dynamic evaluation of YANG at run-time.
All event calls and data go from “provider” to a “consumer” through a central
datastore with MD-SAL. Additionally with the MD-SAL model service lookup was
implemented so that it wasn’t hardwired in ODL which consumers were connected to
which producers.

####OSGI(Open Services Gateway initiative)

OSGi的用处在于“模块化”和“热插拔”。模块化包括模块化、版本化和面向服务的设计。
热插拔也就是说模块/bundle的热插拔，它可以实现更新和升级模块/bundle（即系统的一部分）
而无需重启整个系统。
推荐[这篇文章](http://www.cnblogs.com/Mainz/p/3548396.html)

This framework in the backend of OpenDayLight allows dynamically loading bundles
and packaged Jar files, and binding bundles together for information exchange.


####maven

OpenDayLight uses Maven for easier build automation. Maven uses pom.xml (Project
Object Model for this bundle) to script the dependencies between bundles and also 
to describe what bundles to load on start.

要素
* bundle
* maven archetype
* [maven-bundle-plugin](http://felix.apache.org/site/apache-felix-maven-bundle-plugin-bnd.html)
* pom.xml


[maven tutorial_01](http://blog.csdn.net/ictcamera/article/details/38457567)

####Karaf

[Karaf](http://karaf.apache.org/) is a small OSGi based runtime which provides a lightweight container for loading different modules.

[karaf tutorial](http://karaf.apache.org/manual/latest/quick-start.html)

[karaf hello world](http://www.cnblogs.com/lcchuguo/p/3995286.html)

[opendaylight_karaf_0](https://wiki.opendaylight.org/view/CrossProject:Helium_Release_Vehicle_Brainstorming:Pure_Karaf)
[opendaylight_karaf_1](https://wiki.opendaylight.org/view/Karaf_Distribution_Folder_and_File_Guide)
[opendaylight_karaf_2](https://wiki.opendaylight.org/view/Karaf:Hands_On_Guide)

####Java Interfaces

Java Interfaces are used for event listening, specifications and forming
patterns. This is the main way in which specific bundles implement call-back
functions for events and also to indicate awareness of specific state.

####Life of a packet

<img src="{{ IMAGE_PATH }}/opendaylight/life-in-packet.png" alt="life of packet"
title="opendaylight packet" />

In OpenDaylight, the SAL is in charge of all plumbing between the applications
and the underlying plugins. Here is an illustration of the life of a packet.

    A packet arriving at Switch1 will be sent to the appropriate plugin managing the switch
    The plugin will parse the packet, generate an event for SAL
    SAL will dispatch the packet to the modules listening for DataPacket
    Module handles packet and sends packet_out through IDataPacketService
    SAL dispatches the packet to the modules listening for DataPacket
    OpenFlow message sent to appropriate switch

###参考

https://wiki.opendaylight.org/view/OpenDaylight_Controller:Architectural_Principles

http://sdntutorials.com/opendaylight-netconf-restconf-and-yang/

http://sdntutorials.com/what-is-netconf/

http://sdntutorials.com/what-is-restconf/

http://dtucker.co.uk/work/netconf-yang-restconf-and-netops-in-an-sdn-world.html

http://stackoverflow.com/questions/1612120/osgi-what-are-the-differences-between-apache-felix-and-apache-karaf

###附录

[Java API](https://jenkins.opendaylight.org/controller/job/controller-verify-stable-helium/ws/target/apidocs/index.html)

[Openflow Protocol library](https://wiki.opendaylight.org/view/Openflow_Protocol_Library:Main#Public_API)


[学习 openvswitch 使用](http://www.chenshake.com/based-on-openflow-practices-open-vswitch/)[这里](http://www.sdnap.com/sdnap-post/3520.html)

[学习 minine](https://github.com/mininet/mininet/wiki/Documentation)

[APP 实例](https://wiki.opendaylight.org/view/OpenDaylight_Controller:Hydrogen_Developer_Guide:MD-SAL_App_Tutorial)


必读代码 sal

首先创建一个 mvn 工程，然后创建 freature 绑定到这个工程。这样这个工程就成为一个特性。

###concept

container
Global
instance

LLDPs 
BDDPs
