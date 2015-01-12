---
kind: article
created_at: 2013-08-06 19:29 EET
title: Discovery in Software-Defined Networks
tags:
  - networking
  - openflow
  - sdn
---

*Discovery* can mean a couple of things in a software-defined network including, but is not limited to, the discovery of the 1) switches, 2) links, and 3) hosts. In this blog post I will briefly describe the basics of the underlying plumbing for these three *discovery* tasks. Note that I will assume that the OpenFlow is employed as the south-bound protocol.

Discovery of the Switches
=========================

So how does a controller get to know the switches in its control domain and their features? A switch is initially configured with a *master* controller IP address and a set of *slave* controller IP addresses. While the most recent OpenFlow specifications allow a switch to simultaneously connect to multiple controllers, for simplicity, in this post I will assume that a switch first tries to connect to the master controller and if the connection fails, picks one of the slave controllers -- that is, a switch is connected to a single controller at a time.

When the switch establishes a (TCP) connection to a controller, controller sends a *feature request* message to the switch and waits for a reply. When the reply reaches to the controller, controller gets informed about the features provided by the switch, for instance, the datapath ID (i.e., DPID), list of ports, etc.

Discovery of the Links
=========================

When a switch connects to a controller, controller periodically (e.g., every 5 seconds) commands the switch to flood LLDP (Link Layer Discovery Protocol) and BDDP (Broadcast Domain Discovery Protocol) packets through all of its ports. A discovery protocol packet typically contains the DPID of the sender along with the port of the switch that the message originates from. The reserved set of destination MAC addresses and `ethertype`s used by the discovery protocol packets lets the controller to differentiate them with the other data packets. LLDP is used to discover direct links between switches and BDDP is used to discover the switches in the same broadcast domain. LLDP and BDDP differ from each other by the `ethertype`s they use.

![SDN Topology](network.jpg)

Consider the above network topology, where `s1`, `s2`, and `s3` are connected to a controller and `s4` is a non-OpenFlow switch. Further, each switch is connected to a host. Note that the port numbers are denoted by the numbers at the endpoints of the links. After controller connections to `s1`, `s2`, `s3` get established, the controller commands all three switches to flood LLDP packets. Consequently, LLDP packet sent by `p1` of `s1` (i.e., 1st port of the 1st switch) reaches to `p1` of `s3` and `s3` forwards this packet to the controller. Since the LLDP packet sent by `s1` is actuated by the controller itself, upon receiving the LLDP packet from `s3`, controller discovers that there is a link from `p1` of `s1` to `p1` of `s3`. Likewise, the LLDP packet sent by `p2` of `s2` reaches to `p2` of `s1` and the controller learns that there is a link from `s1` to `s2` as well, and so on. Proceeding in this way the controller eventually figures out all the *direct* links in the topology -- we will come to the case for `s4`. Note that upon receiving an LLDP packet, controller does not forward these packets any further, LLDP packets just acknowledge the controller about the direct links between the switches.

Since `s4` is a non-OpenFlow switch, it does not flood any LLDP packets and even if it would do so, since `s4` is not known by the controller, the LLDP packets originate from `s4` to the OpenFlow controler's domain will not mean anything to the controller. On the other hand topology tells us that `p1` of `s2` is reachable by `p2` of `s3`, in other words, they are in the same *broadcast domain*, but they do not have a direct link between each other. In this case, LLDP and BDDP packets flooded by `s2` and `s3` reaches to the non-OpenFlow switch `s4`. Since LLDP packets are used for direct link discoveries LLDPs get dropped immediatly at the `s4`. However, BDDP packets are broadcasted by non-OpenFlow switches and they make their way to `s2` and `s3`. By this way the controller discovers that there is an indirect connection between `p1` of `s2` and `p2` of `s3` and these switches are in the same broadcast domain.

Discovery of the Hosts
======================

Using a combination of LLDP and BDDP packets, controller discovers the direct and indirect connections between the switches. Further, controller also keeps an eye on the liveliness of the connections regularly with periodical checks. Equipped with the knowledge of the network topology, controller can determine the shortest path from a source switch port to a destination switch port. But how does it discover the hosts? That is, how does the controller discover the *attachment point(s)* of a host?

Considering the previously illustrated network topology, let's assume that `h1` pings `h2`. When ping request reaches to `s1`, since there were no previously installed rule(s) for the incoming flow, `s1` will forward the first packet of the flow to the controller. This is the first time the controller observes a message originating from `h1`. Hence, the controller discovers that `p3` of `s1` is an attachment point for `h1`. But the controller still does not have any knowledge on the attachment points for the destination `h2`. Thus, the controller tells `s1` to flood the packet. Due to flooding, the ping request reaches to `s2` and `s3`. For simplicity, let's assume that the flood first arrived to `s2`. Since `s2` does not have any corresponding rules for the flow, `s2` forwards the packet to the controller. Controller still does not know the attachment point of `h2` and hence commands to `s2` to flood the packet again. After this second flood, the ping request reaches to `h2` and `h2` replies back. Upon `s2` receiving `h2`s reply from `p3`, it forwards the reply to the controller. Now the controller discovers that `p3` of `s2` is an attachment point for `h2`. Since controller has previously discovered `h1`s attachment point, it knows that the reply needs to be routed from `p3` to `p2` of `s2` and then from `p2` to `p3` of `s1`. Consequently, controller installs the appropriate rules to the switches `s1` and `s2`, and finally commands `s2` to send the ping reply from `p2`.

Hosts `h1`, `h2` and `h3` all have a single attachment point from the point of view of the controller. Say this time `h1` pings `h4`. Since the controller does not know the attachment point of `h4`, it will repeatedly flood the request until it makes its way to `s4` and then `h4`. `h4`s reply might reach to the controller network either from `p1` of `s2` or `p2` of `s3`. Further, each packet originating from `h4` can be routed across different paths by `s4` too. To handle such cases, controllers generally assign multiple attachment points for `h4`.
