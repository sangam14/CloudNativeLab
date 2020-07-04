# An Introduction to Networking with Docker
- Docker follows a unique approach to networking that is very different from the Kubernetes approach. 
- Understanding how Docker works help later in understanding the Kubernetes model, since Docker containers are the fundamental unit of deployment in Kubernetes. 



# Docker Networking Types:- 

When a Docker container launches, the Docker engine assigns it a network interface with an IP address, a default gateway, and other components,
such as a routing table and DNS services.  By default, all addresses come from the same pool, and all containers on the same host can communicate with one another.
We can change this by defining the network to which the container should connect, either by creating a custom user-defined network or 
by using a network provider plugin. The network providers are pluggable using drivers. We connect a Docker container to a particular network byusing the `--net` switch when launching it. 
The following command launches a container from the busybox image and joins it to the host network. This container prints its IP address and then exits.

```
docker run --rm --net=host busybox ip addr

```



- Docker offers five network types, each with a different capacity for communication with other network entities.

    -  Host Networking: The container shares the same IP address and network namespace as that of the host. 
Services running inside of this container have the same network capabilities as services running directly on the host

    -  Bridge Networking: The container runs in a private network internal to the host. 
    Communication is open to other containers in the same network. Communication with services outside of the host goes through network address translation 
    (NAT) before exiting the host.  (This is the default mode of networking when the --net option isn't specified)
    - Custom bridge network: This is the same as Bridge Networking but uses a bridge explicitly created for this (and other) containers.
   An example of how to use this would be a container that runs on an exclusive "database" bridge network. 
   Another container can have an interface on the default bridge and the database bridge, enabling it to communicate with both networks.
   - Container-defined Networking: A container can share the address and network configuration of another container.
   This type enables process isolation between containers, where each container runs one service but where services can still communicate 
    with one another on the localhost address.E. No networking: This option disables all networking for the container
    
# Host Networking

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/rancher_host_net.png)

The host mode of networking allows the Docker container to share the same IP address as that of the host and disables 
the network isolation otherwise provided by network namespaces. The container’s network stack is mapped directly to the host’s network stack.
All interfaces and addresses on the host are visible within the container, and all communication possible to or from the host is possible to or from the container.
If you run the command ip addr on a host (or ifconfig -a if your host doesn’t have the ipcommand available), 
you will see information about the network interfaces.



   
   ```
    
    $ ip address
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN 
    link/ether 02:42:bf:42:ab:9f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
21079: eth0@if21080: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 22:8a:64:31:ae:4c brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.28/23 scope global eth0
       valid_lft forever preferred_lft forever
21083: eth1@if21084: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue state UP 
    link/ether 02:42:ac:12:00:09 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.9/16 scope global eth1
       valid_lft forever preferred_lft forever
[node1] (local) root@192.168.0.28 ~
$ 
```

If you run the same command from a container using host networking, you will see the same information.


```
docker run --rm --net=host busybox ip addr
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
91f30d776fb2: Pull complete 
Digest: sha256:9ddee63a712cea977267342e8750ecbc60d3aab25f04ceacfa795e6fce341793
Status: Downloaded newer image for busybox:latest
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
2: docker0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue 
    link/ether 02:42:bf:42:ab:9f brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.1/16 brd 172.17.255.255 scope global docker0
       valid_lft forever preferred_lft forever
21079: eth0@if21080: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 22:8a:64:31:ae:4c brd ff:ff:ff:ff:ff:ff
    inet 192.168.0.28/23 scope global eth0
       valid_lft forever preferred_lft forever
21083: eth1@if21084: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:12:00:09 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.9/16 scope global eth1
       valid_lft forever preferred_lft forever
[node1] (local) root@192.168.0.28 ~


```

