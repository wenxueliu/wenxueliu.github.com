The Network Configuration Protocol, NETCONF, is an IETF network management
protocol. It was developed in the NETCONF working group and published in
December 2006 as [RFC 4741](https://tools.ietf.org/html/rfc4741) and later
revised in June 2011 and published as [RFC 6241](https://tools.ietf.org/html/rfc6241).
The NETCONF protocol specification is an Internet Standards Track document.

NETCONF provides mechanisms to install, manipulate, and delete the configuration
of network devices. Its operations are realized on top of a simple Remote
Procedure Call (RPC) layer. The NETCONF protocol uses an Extensible Markup
Language (XML) based data encoding for the configuration data as well as the
protocol messages. This in turn is realized on top of the transport protocol.
(from Wikipedia)

###Feature

    Configuration system capable of two phase commit
    Configuration APIs and SPIs exposed via JMX config beans
    Runtime statistics and other data exposed via JMX runtime beans
    yang code generator to generate JMX, configuration interfaces and skeletons of Modules and ModuleFactories
    netconf server (plain TCP and ssh) is used as config-subsystem endpoint, wraps JMX config and runtime beans into model driven XML representation
    netconf client with advanced capabilities like reconnection strategies and capability testing
    configuration pusher and persister that delivers initial or current configuration, automatic persisting of changes after netconf commit
    netconf monitoring capability in netconf server
    many low level Java APIs wrapped with yang models and configuration factories, like executor framework, netty threadgroups, logback
    shutdown management via netconf RPC

###ARCH

NetConf tests are needed for testing northbound connectivity to our controller
and southbound connectivity to an external netconf device. 

  South Bound: Server
  North Bound: Cient


[DOCS](http://www.netconfcentral.org/netconf_docs)
[Feature](https://wiki.opendaylight.org/view/Config_Netconf:Hydrogen_Release_Notes)
[Example](https://wiki.opendaylight.org/view/OpenDaylight_Controller:Config:Examples:Netconf:Example_Configuration)
[Component Map](https://wiki.opendaylight.org/view/OpenDaylight_Controller:Netconf:Component_Map)



待读

https://wiki.opendaylight.org/view/OpenDaylight_Controller:Netconf:Design
https://wiki.opendaylight.org/view/OpenDaylight_Controller:Config:Examples:Netconf:Manual_netopeer_installation
https://wiki.opendaylight.org/view/OpenDaylight_Controller:Netconf:Component_Map
https://wiki.opendaylight.org/view/CrossProject:Integration_Group:NetConf_Test
https://wiki.opendaylight.org/view/CrossProject:Integration_Group:NetConf_SB
https://wiki.opendaylight.org/view/OpenDaylight_Controller:Netconf:Testtool
http://www.netconfcentral.org/netconf_docs

https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Toaster_Step-By-Step
https://wiki.opendaylight.org/view/OpenDaylight_Controller:MD-SAL:Toaster_Tutorial
