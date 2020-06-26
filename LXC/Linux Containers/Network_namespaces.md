---
layout: default
title: Network namespaces
parent: LXC Hands-On Workshop
nav_order: 9
---


# Network namespaces

- Network namespaces provide isolation of the networking resources, such as network devices, addresses, routes, and firewall rules. This effectively creates a logical copy of the network stack, allowing multiple processes to listen on the same port from multiple namespaces. This is the foundation of networking in LXC and there are quite a lot of other use cases where this can come in handy.

- The `iproute2` package provides very useful userspace tools that we can use to experiment with the network namespaces, and is installed by default on almost all Linux systems.

- There's always the default network namespace, referred to as the root namespace, where all network interfaces are initially assigned. To list the network interfaces that belong to the default
namespace run the following command:

```
root@server:~# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
    link/ether 0e:d5:0e:b0:a3:47 brd ff:ff:ff:ff:ff:ff
root@server:~#

```
In this case, there are two interfaces – `lo` and `eth0`.

To list their configuration, we can run the following:

```
root@server:~# ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
valid_lft forever preferred_lft forever
inet6 ::1/128 scope host
    valid_lft forever preferred_lft forever
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP group default qlen 1000
    link/ether 0e:d5:0e:b0:a3:47 brd ff:ff:ff:ff:ff:ff
inet 10.1.32.40/24 brd 10.1.32.255 scope global eth0
valid_lft forever preferred_lft forever
inet6 fe80::cd5:eff:feb0:a347/64 scope link
    valid_lft forever preferred_lft forever
root@server:~#

```
Also, to list the routes from the root network namespace, execute the following:

```
root@server:~# ip r s
default via 10.1.32.1 dev eth0
10.1.32.0/24 dev eth0  proto kernel  scope link  src 10.1.32.40
root@server:~#

```

Let's create two new network namespaces called `ns1` and `ns2` and list them:

```
root@server:~# ip netns add ns1
root@server:~# ip netns add ns2
root@server:~# ip netns
ns2
ns1
root@server:~#
```

Now that we have the new network namespaces, we can execute commands inside them:

```
root@server:~# ip netns exec ns1 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
root@server:~#

```

The preceding output shows that in the ns1 namespace, there's only one network interface, the loopback - `lo` interface, and it's in a DOWN state.

We can also start a new bash session inside the namespace and list the interfaces in a similar way:

```
root@server:~# ip netns exec ns1 bash
root@server:~# ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default
 link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
root@server:~# exit
root@server:~#

```

- This is more convenient for running multiple commands than specifying each, one at a time. The two network namespaces are not of much use if not connected to anything, so let's connect them to each other. To do this we'll use a software bridge called Open vSwitch.

- Open vSwitch works just as a regular network bridge and then it forwards frames between virtual ports that we define. Virtual machines such as KVM, Xen, and LXC or Docker containers can then be connected to it.

- Most Debian-based distributions such as Ubuntu provide a package, so let's install that:


```
root@server:~# apt-get install -y openvswitch-switch
root@server:~#
```

This installs and starts the Open vSwitch daemon. Time to create the bridge; we'll name it OVS-1:

```
root@server:~# ovs-vsctl add-br OVS-1
root@server:~# ovs-vsctl show
0ea38b4f-8943-4d5b-8d80-62ccb73ec9ec
Bridge "OVS-1"
    Port "OVS-1"
        Interface "OVS-1"
            type: internal
ovs_version: "2.0.2"
root@server:~#

```
# Note
```
If you would like to experiment with the latest version of Open vSwitch, you can download the source code from http://openvswitch.org/download/ and compile it.
```
The newly created bridge can now be seen in the root namespace:

```
root@server:~# ip a s OVS-1
4: OVS-1: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN group default
link/ether 9a:4b:56:97:3b:46 brd ff:ff:ff:ff:ff:ff
inet6 fe80::f0d9:78ff:fe72:3d77/64 scope link
   valid_lft forever preferred_lft forever
root@server:~#
```

In order to connect both network namespaces, let's first create a virtual pair of interfaces for each namespace:

```
root@server:~# ip link add eth1-ns1 type veth peer name veth-ns1
root@server:~# ip link add eth1-ns2 type veth peer name veth-ns2
root@server:~#


```
The preceding two commands create four virtual interfaces `eth1-ns1`, `eth1-ns2` and `veth-ns1`, `veth-ns2`. The names are arbitrary.

To list all interfaces that are part of the root network namespace, run:

```
root@server:~# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
link/ether 0e:d5:0e:b0:a3:47 brd ff:ff:ff:ff:ff:ff
3: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default
link/ether 82:bf:52:d3:de:7e brd ff:ff:ff:ff:ff:ff
4: OVS-1: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default
link/ether 9a:4b:56:97:3b:46 brd ff:ff:ff:ff:ff:ff
5: veth-ns1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether 1a:7c:74:48:73:a9 brd ff:ff:ff:ff:ff:ff
6: eth1-ns1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether 8e:99:3f:b8:43:31 brd ff:ff:ff:ff:ff:ff
7: veth-ns2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether 5a:0d:34:87:ea:96 brd ff:ff:ff:ff:ff:ff
8: eth1-ns2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether fa:71:b8:a1:7f:85 brd ff:ff:ff:ff:ff:ff
root@server:~#


```

Let's assign the eth1-ns1 and eth1-ns2 interfaces to the ns1 and ns2 namespaces:

```
root@server:~# ip link set eth1-ns1 netns ns1
root@server:~# ip link set eth1-ns2 netns ns2
```

Also, confirm they are visible from inside each network namespace:
```
root@server:~# ip netns exec ns1 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
6: eth1-ns1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether 8e:99:3f:b8:43:31 brd ff:ff:ff:ff:ff:ff
root@server:~#
root@server:~# ip netns exec ns2 ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
8: eth1-ns2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether fa:71:b8:a1:7f:85 brd ff:ff:ff:ff:ff:ff
root@server:~#

```
Notice, how each network namespace now has two interfaces assigned – `loopback` and `eth1-ns*`.

If we list the devices from the root namespace, we should see that the interfaces we just moved to ns1 and ns2 namespaces are no longer visible:

```
root@server:~# ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
2: eth0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 9001 qdisc pfifo_fast state UP mode DEFAULT group default qlen 1000
link/ether 0e:d5:0e:b0:a3:47 brd ff:ff:ff:ff:ff:ff
3: ovs-system: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default
link/ether 82:bf:52:d3:de:7e brd ff:ff:ff:ff:ff:ff
4: OVS-1: <BROADCAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UNKNOWN mode DEFAULT group default
link/ether 9a:4b:56:97:3b:46 brd ff:ff:ff:ff:ff:ff
5: veth-ns1: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether 1a:7c:74:48:73:a9 brd ff:ff:ff:ff:ff:ff
7: veth-ns2: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
link/ether 5a:0d:34:87:ea:96 brd ff:ff:ff:ff:ff:ff
root@server:~#


```

It's time to connect the other end of the two virtual pipes, the veth-ns1 and veth-ns2 interfaces to the bridge:
```
root@server:~# ovs-vsctl add-port OVS-1 veth-ns1
root@server:~# ovs-vsctl add-port OVS-1 veth-ns2
root@server:~# ovs-vsctl show
0ea38b4f-8943-4d5b-8d80-62ccb73ec9ec
Bridge "OVS-1"
    Port "OVS-1"
        Interface "OVS-1"
            type: internal
    Port "veth-ns1"
        Interface "veth-ns1"
    Port "veth-ns2"
        Interface "veth-ns2"
ovs_version: "2.0.2"
root@server:~#


```
From the preceding output, it's apparent that the bridge now has two ports, veth-ns1 and veth-ns2.

The last thing left to do is bring the network interfaces up and assign IP addresses:
```
root@server:~# ip link set veth-ns1 up
root@server:~# ip link set veth-ns2 up
root@server:~# ip netns exec ns1 ip link set dev lo up
root@server:~# ip netns exec ns1 ip link set dev eth1-ns1 up
root@server:~# ip netns exec ns1 ip address add 192.168.0.1/24 dev eth1-ns1
root@server:~# ip netns exec ns1 ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
   valid_lft forever preferred_lft forever
inet6 ::1/128 scope host
   valid_lft forever preferred_lft forever
6: eth1-ns1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
link/ether 8e:99:3f:b8:43:31 brd ff:ff:ff:ff:ff:ff
inet 192.168.0.1/24 scope global eth1-ns1
   valid_lft forever preferred_lft forever
inet6 fe80::8c99:3fff:feb8:4331/64 scope link
   valid_lft forever preferred_lft forever
root@server:~#


```

Similarly for the ns2 namespace:

```
root@server:~# ip netns exec ns2 ip link set dev lo up
root@server:~# ip netns exec ns2 ip link set dev eth1-ns2 up
root@server:~# ip netns exec ns2 ip address add 192.168.0.2/24 dev eth1-ns2
root@server:~# ip netns exec ns2 ip a s
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default
link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
inet 127.0.0.1/8 scope host lo
   valid_lft forever preferred_lft forever
inet6 ::1/128 scope host
   valid_lft forever preferred_lft forever
8: eth1-ns2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
link/ether fa:71:b8:a1:7f:85 brd ff:ff:ff:ff:ff:ff
inet 192.168.0.2/24 scope global eth1-ns2
   valid_lft forever preferred_lft forever
inet6 fe80::f871:b8ff:fea1:7f85/64 scope link
   valid_lft forever preferred_lft forever
root@server:~#


```
