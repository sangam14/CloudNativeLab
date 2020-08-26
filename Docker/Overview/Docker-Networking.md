---
layout: default
title: Docker Network
parent: Docker Fundamental
nav_order: 6
---

# What is a Docker network?

- a network is a linkage system that allows computers and other hardware devices to communicate. A Docker network is the same thing. It is a linkage system that 
allows Docker containers to communicate with each other on the same Docker host, or with containers, computers, and hardware outside of the container's host, 
including containers running on other Docker hosts.

- If you are familiar with the cloud computing analogy of pets versus cattle, you understand the necessity of being able to manage resources at scale.
Docker networks allow you to do just that. They abstract away most of the complexity of networking, delivering easy-to-understand, easy-to-document,
and easy-to-use networks for your containerized apps. The Docker network is based on a standard, created by Docker, called the Container Network Model (CNM). 
There is a competing networking standard, created by CoreOS, called the Container Network Interface (CNI). The CNI standard has been adopted by several projects, 
most notably Kubernetes, and arguments can be made to support its use. we will focus our attention on the CNM standard from Docker.

- The CNM has been implemented by the libnetwork project, and you can learn more about that project by following the link in the references for this section.
The CNM implementation, written in Go, is made up of three constructs: the sandbox, the endpoint, and the network. The sandbox is a network namespace.
Each container has its own sandbox. It holds the configuration of the container's network stack. This includes its routing tables, interfaces, and DNS settings 
for IP and MAC addresses. The sandbox also contains the network endpoints for the container. Next, the endpoints are what join the sandbox to networks.
Endpoints are essentially network interfaces, such as eth0. A container's sandbox may have more than one endpoint, but each endpoint will connect to only a 
single network. Finally, a network is a collection of connected endpoints, which allow communication between connections. Every network has a name, an address space, an ID, 
and a network type.

- Libnetwork is a pluggable architecture that allows network drivers to implement the specifics for the components we just described. Each network type has
its own network driver. Docker provides built-in drivers. These default, or local, drivers include the bridge driver and the overlay driver. In addition
to the built-in drivers, libnetwork supports third-party-created drivers. These drivers are referred to as remote drivers. Some examples of remote drivers 
include Calico, Contiv, and Weave.

- You now know a little about what a Docker network is, and after reading these details, you might be thinking, where's the easy that he talked about? 
Hang in there. now we are going to start discussing how easy it is for you to create and use Docker networks. As with Docker volume, the network commands represent their own management category.
As you would expect, the top-level management command for network is as follows:


```
# Docker network managment command
docker network

```

The subcommands available in the network management group include the following:

```
# Docker network management subcommands
docker network connect           # Connect a container to a network
docker network create            # Create a network
docker network disconnect        # Disconnect a container from a network
docker network inspect           # Display network details
docker network ls                # List networks
docker network rm                # Remove one or more networks
docker network prune             # Remove all unused networks

```
Let's now take a look at the built-in or local network drivers.

- Check out the following links for more information:
    - [Pets versus cattle talk slide-deck](https://www.slideshare.net/randybias/architectures-for-open-and-scalable-clouds)
    - [Libnetwork project](https://github.com/docker/libnetwork)
    - [Libnetwork design](https://github.com/docker/libnetwork/blob/master/docs/design.md)
    - [Calico network driver](https://www.projectcalico.org/)
    - [Contiv network driver](http://contiv.github.io/)
    - [Weave network driver](https://www.weave.works/docs/net/latest/overview/)
    
    
## Built-in (local) Docker networks

The out-of-the-box install of Docker includes a few built-in network drivers. These are also known as local drivers. The two most commonly used drivers
are the bridge network driver and the overlay network driver. Other built-in drivers include none, host, and MACVLAN. Also, without your creating networks,
your fresh install will have a few networks pre-created and ready to use. Using the network ls command, we can easily see the list of pre-created networks
available in the fresh installation:

```
# view current networks
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
e1e07875d2ff        bridge              bridge              local
a9cb8634faea        host                host                local
03b03a3a7725        none                null                local
```
- In this list, you will notice that each network has its unique ID, a name, a driver used to create it (and that controls it), and a network scope.
Don't confuse a scope of local with the category of driver, which is also local. The local category is used to differentiate the driver's
origin from third-party drivers that have a category of remote. A scope value of local indicates that the limit of communication for the network 
is bound to within the local Docker host. To clarify, if two Docker hosts, H1 and H2, both contain a network that has the scope of local,
containers on H1 will never be able to communicate directly with containers on H2, even if they use the same driver and the networks have the same name. 
The other scope value is swarm.

- The pre-created networks that are found in all deployments of Docker are special in that they cannot be removed. It is not necessary to attach containers 
to any of them, but attempts to remove them with the docker network rm command will always result in an error.

- There are three built-in network drivers that have a scope of local: bridge, host, and none. The host network driver leverages the networking stack of 
the Docker host, essentially bypassing the networking of Docker. All containers on the host network are able to communicate with each other through the 
host's interfaces. A significant limitation to using the host network driver is that each port can only be used by a single container.
That is, for example, you cannot run two nginx containers that are both bound to port 80. As you may have guessed because the host driver leverages the network 
of the host it is running on, each Docker host can only have one network using the host driver:

```

$ docker container run -d --network host --name container-host-net alpine tail -f /dev/null
$ docker container exec -it container-host-net ifconfigdocker0   Link encap:Ethernet  HWaddr 02:42:EC:AA:78:F4  
          inet addr:172.17.0.1  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

eth0      Link encap:Ethernet  HWaddr C6:D6:B4:6F:9D:CB  
          inet addr:192.168.0.13  Bcast:0.0.0.0  Mask:255.255.254.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:16 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:1296 (1.2 KiB)  TX bytes:0 (0.0 B)

eth1      Link encap:Ethernet  HWaddr 02:42:AC:12:00:07  
          inet addr:172.18.0.7  Bcast:0.0.0.0  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:7377 errors:0 dropped:0 overruns:0 frame:0
          TX packets:3165 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:3755418 (3.5 MiB)  TX bytes:3490929 (3.3 MiB)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:32 errors:0 dropped:0 overruns:0 frame:0
          TX packets:32 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:4230 (4.1 KiB)  TX bytes:4230 (4.1 KiB)


```

Next up, is the null or none network. Using the null network driver creates a network that when a container is connected to it provides a
full network stack but does not configure any interfaces within the container. This renders the container completely isolated. This driver is provided mainly for 
backward-compatibility purposes, and like the host driver, only one network of the null type can be created on a Docker host:

```
 $ docker container run -d --network none --name container-null-net alpine tail -f /dev/null
965963f8872ebc0373c8b25d778d606d59af3620623b3e2499b2b84879f6d023
$  docker container exec -it container-null-net ifconfiglo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```
The third network driver with a scope of local is the bridge driver. Bridge networks are the most common type. Any containers attached to the same bridge 
network are able to communicate with one another. A Docker host can have more than one network created with the bridge driver. However,
containers attached to one bridge network are unable to communicate with containers on a different bridge network, even if the networks are on 
the same Docker host. Note that there are slight feature differences between the built-in bridge network and any user-created bridge networks. 
It is best practice to create your own bridge networks and utilize them instead of the using the built-in bridge network.  Here is an example of running a container using a bridge network:

```
docker container run -d --network bridge --name container-bridge-net alpine tail -f /dev/null
e9046e11c8b77a4242310e4affedea8dde0f735974b86c4068e24648c3841967
[node1] (local) root@192.168.0.13 ~
$ docker container exec -it container-bridge-net ifconfigeth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02  
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback  
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

[node1] (local) root@192.168.0.13 ~
$ 


```
- In addition to the drivers that create networks with local scope, there are built-in network drivers that create networks with swarm scope.
Such networks will span all the hosts in a swarm and allow containers attached to them to communicate in spite of running on different Docker hosts.
As you probably have surmised, use of networks that have swarm scope requires Docker swarm mode. In fact, when you initialize a Docker host into swarm mode,
a special new network is created for you that has swarm scope. This swarm scope network is named ingress and is created using the built-in overlay driver.
This network is vital to the load balancing feature of swarm mode that saw used in the Accessing container applications Docker Swarm. There's also a new bridge network created in theswarm init, named docker_gwbridge. This network is used by swarm to communicate outward, 
kind of like a default gateway.  Here are the default built-in networks found in a new Docker swarm:


```
$ docker swarm init 
$ docker network ls 
NETWORK ID          NAME                DRIVER              SCOPE
45e4af6aa9b0        bridge              bridge              local
04c14b6ebd77        docker_gwbridge     bridge              local
1a177dc8906e        host                host                local
8u5q4nf9t6jb        ingress             overlay             swarm
7f6bed45eaf0        none                null                local

```

- Using the overlay driver allows you to create networks that span Docker hosts. These are layer 2 networks. There is a lot of network plumbing 
that gets laid down behind the scenes when you create an overlay network. Each host in the swarm gets a network sandbox with a network stack. Within that 
sandbox, a bridge is created and named br0. Then, a VXLAN tunnel endpoint is created and attached to bridge br0. Once all of the swarm hosts have the
tunnel endpoint created, a VXLAN tunnel is created that connects all of the endpoints together. This tunnel is actually what we see as the overlay network. 
When containers are attached to the overlay network, they get an IP address assigned from the overlay's subnet, and all communications between containers on 
that network are carried out via the overlay. Of course, behind the scenes that communication traffic is passing through the VXLAN endpoints, going across 
the Docker hosts network, and any routers connecting the host to the networks of the other Docker hosts. But, you never have to worry about all the 
behind-the-scenes stuff. Just create an overlay network, attach your containers to it, and you're golden.

- The next local network driver that we're going to discuss is called MACVLAN. This driver creates networks that allow containers to each have their own 
IP and MAC addresses, and to be attached to a non-Docker network. What that means is that in addition to the container-to-container communication you get 
with bridge and overlay networks, with MACVLAN networks you also are able to connect with VLANs, VMs, and other physical servers. Said another way, 
the MACVLAN driver allows you to get your containers onto existing networks and VLANs. A MACVLAN network has to be created on each Docker host where you will
run containers that need to connect to your existing networks. What's more, you will need a different MACVLAN network created for each VLAN you want containers
to connect to. While using MACVLAN networks sounds like the way to go, there are two important challenges to using it. First, you have to be very careful about
the subnet ranges you assign to the MACVLAN network. Containers will be assigned IPs from your range without any consideration of the IPs in use elsewhere. If 
you have a DHCP system handing out IPs that overlap with the range you gave to the MACVLAN driver, it can easily cause duplicate IP scenarios. The second challenge 
is that MACVLAN networks require your network cards to be configured in promiscuous mode. This is usually frowned upon in on-premise networks but is pretty
much forbidden in cloud-provider networks such as AWS and Azure, so the MACVLAN driver will have very limited use cases.

- Check out these links for more information:
  - Excellent, in-depth Docker article for Docker networking: https://success.docker.com/article/networking
  - Networking with Overlay Networks: https://docs.docker.com/network/network-tutorial-overlay/
  - Using MACVLAN networks: https://docs.docker.com/v17.12/network/macvlan/
  
  
# Third-party (remote) network drivers

- in addition to the built-in, or local, network drivers provided by Docker, the CNM supports community- and vendor-created network drivers. Some examples 
of these third-party drivers include Contiv, Weave, Kuryr, and Calico. One of the benefits of using one of these third-party drivers is that they fully 
support deployment in cloud-hosted environments, such as AWS. In order to use these drivers, they need to be installed in a separate installation 
step for each of your Docker hosts. Each of the third-party network drivers brings their own set of features to the table.
Here is the summary description of these drivers as shared by Docker in the reference architecture document:
![]()


Although each of these third-party drivers has its own unique installation, setup, and execution methods, the general steps are similar. 
First, you download the driver, then you handle any configuration setup, and finally you run the driver. These remote drivers typically do not 
require swarm mode and can be used with or without it. As an example, let's take a deep-dive into using the weave driver.
To install the weave network driver, issue the following commands on each Docker host:

```
# Install the weave network driver plug-in
sudo curl -L git.io/weave -o /usr/local/bin/weave
sudo chmod a+x /usr/local/bin/weave
# Disable checking for new versions
export CHECKPOINT_DISABLE=1
# Start up the weave network
weave launch [for 2nd, 3rd, etc. optional hostname or IP of 1st Docker host running weave]
# Set up the environment to use weave
eval $(weave env)

```

The preceding steps need to be completed on each Docker host that will be used to run containers that will communicate with each other over the weave network. The launch command can provide the hostname or IP address of the first Docker host, which was set up and already running the weave network, to peer with it so that their containers can communicate. For example,
if you have set up `node01` with the weave network when you start up weave on `node02`, you would use the following command:

```
# Start up weave on the 2nd node
weave launch node01

```
Alternatively, you can connect new (Docker host) peers using the connect command, executing it from the first host configured.
To add node02 (after it has weave installed and running), use the following command:

```
# Peer host node02 with the weave network by connecting from node01
weave connect node02
```
You can utilize the weave network driver without enabling swarm mode on your hosts. Once weave has been installed and started, and the peers (other Docker hosts) have been connected, your containers will automatically utilize the weave network and be able to communicate with each other 
regardless of whether they are on the same Docker host or different ones.

The weave network shows up in your network list just like any of your other networks:

```



```



    
    
