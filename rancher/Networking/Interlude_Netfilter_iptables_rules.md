---
layout: default
title: Interlude - Netfilter and iptables rules
parent: Rancher Networking
nav_order: 4
---



# Interlude: Netfilter and iptables rules

- we looked at how Docker handles communication between containers. On a Linux host, the component which handles this is called Netfilter, or more commonly by the command used to configure it: iptables.Netfilter manages the rules that define network communication for the Linux kernel. 
These rules permit, deny, route, modify, and forward packets. It organizes these rules into tables according to their purpose.

- Netfilter manages the rules that define network communication for the Linux kernel. 
These rules permit, deny, route, modify, and forward packets. It organizes these rules into tables according to their purpose. 

# The Filter Table

  - The Filter TableRules in the Filter table control if a packet is allowed or denied.
Packets which are allowed are forwarded whereas packets which are denied are either rejected or silently dropped


# The NAT Table

- These rules control network address translation. They modify the source or destination address for the packet, changing how the kernel routes the packet

# The Mangle Table
 
-  The headers of packets which go through this table are altered, changing the way the packet behaves. Netfilter might shorten the TTL, redirect it to a different address, or change the number of network hops.

# Raw Table

- This table marks packets to bypass the iptables stateful connection tracking. 
Security TableThis table sets the SELinux security context marks on packets. Setting the marks affects how SELinux (or systems that can interpret SELinux security contexts) handle the packets.
The rules in this table set marks on a per-packet or per-connection basis.Netfilter organizes the rules in a table into chains.
Chains are the means by which Netfilter hooks in the kernel intercept packets as they move through processing. Packets flow through one or more chains and 
exit when they match a rule.A rule defines a set of conditions, and if the packet matches those conditions, an action is taken. The universe of actions is diverse,
but examples include:
   - Block all connections originating from a specific IP address. <br>
   - Block connections to a network interface. <br>
   - Allow all HTTP/HTTPS connections.<br>
   - Block connections to specific ports.The action that a rule takes is called a target, and represents the decision to accept, drop, or forward the packet. 
The system comes with five default chains that match different phases of a packet’s journey through processing: PREROUTING, INPUT, FORWARD, OUTPUT,
and POSTROUTING. Users and programs may create additional chains and inject rules into the system chains to forward packets to a custom chain for 
continued processing. 
- This architecture allows the Netfilter configuration to follow a logical structure, with chains representing groups of 
related rules.Docker creates several chains, and it is the actions of these chains that handle communication between containers, the host, and the outside world.
