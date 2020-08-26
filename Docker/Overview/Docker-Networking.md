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
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/docker-driver-network.png)


Although each of these third-party drivers has its own unique installation, setup, and execution methods, the general steps are similar. 
First, you download the driver, then you handle any configuration setup, and finally you run the driver. These remote drivers typically do not 
require swarm mode and can be used with or without it. As an example, let's take a deep-dive into using the weave driver.
To install the weave network driver, issue the following commands on each Docker host:

```
[node1] (local) root@192.168.0.23 ~
$ sudo curl -L git.io/weave -o /usr/local/bin/weave
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
  0     0    0     0    0     0      0      0 --:--:-- --:--:-- --:--:--     0
100   629  100   629    0     0    915      0 --:--:-- --:--:-- --:--:--   915
100 51395  100 51395    0     0  68986      0 --:--:-- --:--:-- --:--:-- 68986
[node1] (local) root@192.168.0.23 ~
$ sudo chmod a+x /usr/local/bin/weave
[node1] (local) root@192.168.0.23 ~
$ export CHECKPOINT_DISABLE=1
[node1] (local) root@192.168.0.23 ~
$ weave launch
2.7.0: Pulling from weaveworks/weave
21c83c524219: Pull complete 
3c1275a4379d: Pull complete 
71ff85185d24: Pull complete 
ba2937042659: Pull complete 
35b26289b9d6: Pull complete 
Digest: sha256:be309a6d90bb8663c479e6873ffc9f6265c40decac64c6b602af5084d851aef6
Status: Downloaded newer image for weaveworks/weave:2.7.0
docker.io/weaveworks/weave:2.7.0
latest: Pulling from weaveworks/weavedb
4e2a4496fae2: Pull complete 
Digest: sha256:836c82b6bf5039c5d240af4c4ef5d2ce50f826a66fceaac370992db1a82b7bb0
Status: Downloaded newer image for weaveworks/weavedb:latest
docker.io/weaveworks/weavedb:latest
Unable to find image 'weaveworks/weaveexec:2.7.0' locally
2.7.0: Pulling from weaveworks/weaveexec
21c83c524219: Already exists 
3c1275a4379d: Already exists 
71ff85185d24: Already exists 
ba2937042659: Already exists 
35b26289b9d6: Already exists 
65c7f7e0a1c9: Pull complete 
48e97ff78ad1: Pull complete 
a2ba25002c0c: Pull complete 
8678f227127c: Pull complete 
Digest: sha256:17e8d712976e574d68181159d62f45449ee1b7376c200a87f2edd48890fc5650
Status: Downloaded newer image for weaveworks/weaveexec:2.7.0
46ec65c48a63b87e507e9f5832b4e1d4fe5bb2d561f935b6c908aeeb306166a0
[node1] (local) root@192.168.0.23 ~
$ eval $(weave env)
[node1] (unknown) root@192.168.0.23 ~
$ 

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
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
0e6e8fc2dcda        bridge              bridge              local
0e38259af941        host                host                local
665ffb6c7c62        none                null                local
34f4a8b1c448        weave               weavemesh           local

```

Let's test out our shiny new network. First, make sure that you have installed the weave driver on all the hosts you want to be connected by following the steps described previously. Make sure that you either use the launch command with node01 as a parameter, or from node01 you use the connect command for each of the additional nodes you are configuring. For this example, my lab servers are named node01 and node02. Let's start with node02:

```
# Install and setup the weave driver
sudo curl -L git.io/weave -o /usr/local/bin/weave
sudo chmod a+x /usr/local/bin/weave
export CHECKPOINT_DISABLE=1
weave launch
eval $(weave env)

```
And, note the following, on ubuntu-node02:

```
# Install and setup the weave driver
sudo curl -L git.io/weave -o /usr/local/bin/weave
sudo chmod a+x /usr/local/bin/weave
export CHECKPOINT_DISABLE=1
weave launch
eval $(weave env)

```
Now, back on ubuntu-node01, note the following:

```
# Bring node02 in as a peer on node01's weave network
weave connect ubuntu-node02

```
Now, let's launch a container on each node. Make sure we name them for easy identification, starting with ubuntu-node01:

```
# Run a container detached on node01
docker container run -d --name app01 alpine tail -f /dev/null

```

Now, launch a container on ubuntu-node02:

```
# Run a container detached on node02
docker container run -d --name app02 alpine tail -f /dev/null


```
Excellent. Now, we have containers running on both nodes. Let's see whether they can communicate. Since we are on node02, we will check there first:

```
# From inside the app02 container running on node02,
# let's ping the app01 container running on node01
docker container exec -it app02 ping -c 4 app01


```

Yeah! That worked. Let's try going the other way:

```
# Similarly, from inside the app01 container running on node01,
# let's ping the app02 container running on node02
docker container exec -it app01 ping -c 4 app02


```

Perfect! We have bi-directional communication. Did you notice anything else? We have name resolution for our app containers (we didn't have to ping by IP only). Pretty nice, right?

- Check out these links for more information:
    - [Installing and using the weave network driver](https://www.weave.works/docs/net/latest/overview/)
    - [Weaveworks weave github repo](https://github.com/weaveworks/weave)
    

# Creating Docker networks


OK, you now know a lot about both the local and the remote network drivers, and you have seen how several of them are created for you when you install Docker and/or initialize swarm mode (or install a remote driver). But, what if you want to create your own networks using some of these drivers? It is really pretty simple. Let's take a look. The built-in help for the network create command looks like this:

```
# Docker network create command syntax
# Usage: docker network create [OPTIONS] NETWORK

```

Probably the most important option is the `--driver` option. This is how we tell Docker which of the pluggable network drivers to use when creating this network. As you have seen, the choice of driver determines the network characteristics. The value you supply to the driver option will be like the ones shown in the DRIVER column of the output from the `docker network ls` command. Some of the possible values are bridge, overlay, and macvlan. Remember that you cannot create additional host or null networks as they are limited to one per Docker host. So far, what might this look like? Here is an example of creating a new overlay network, using mostly defaults for options:

```
# Create a new overlay network, with all default options
docker network create -d overlay defaults-over

```

That works just fine. You can run new services and attach them to your new network. But what else might we want to control in our network? Well, how about the IP space? Yep, and Docker provides options for controlling the IP settings for our networks. This is done using the `--subnet`, `--gateway`, and `--ip-range` optional parameters. So, let's take a look at creating a new network using this options. See Chapter 2, Learning Docker Commands, for how to install jq if you have not done so already:

```
# Create a new overlay network with specific IP settings
docker network create -d overlay \
--subnet=172.30.0.0/24 \
--ip-range=172.30.0.0/28 \
--gateway=172.30.0.254 \
specifics-over
# Initial validation
docker network inspect specifics-over --format '{{json .IPAM.Config}}' | jq


```

Executing the preceding code in my lab looks like this:

```
[manager1] (local) root@192.168.0.19 ~
$ # Create a new overlay network with specific IP settings
[manager1] (local) root@192.168.0.19 ~
$ docker network create -d overlay \
> --subnet=172.30.0.0/24 \
> --ip-range=172.30.0.0/28 \
> --gateway=172.30.0.254 \
> specifics-over
ozgy9b0wge8jhilm3her61oln
[manager1] (local) root@192.168.0.19 ~
$ # Initial validation
[manager1] (local) root@192.168.0.19 ~
$ docker network inspect specifics-over --format '{{json .IPAM.Config}}' | jq
[
  {
    "Subnet": "172.30.0.0/24",
    "IPRange": "172.30.0.0/28",
    "Gateway": "172.30.0.254"
  }
[manager1] (local) root@192.168.0.19 ~

```
Looking over this example, we see that we created a new overlay network using specific IP parameters for the subnet, the IP range, and the gateway. Then, we validated that the network was created with the requested options. Next, we created a service using our new network. Then, we found the container ID for a container belonging to the service and used it to inspect the network settings for the container. We can see that the container was run using an IP address (in this case, 172.30.0.7) from the IP range we configured our network with. Looks like we made it! 

As mentioned, there are many other options available when creating Docker networks,
and I will leave it as an exercise for you to discover them with the docker network create --help command, and to try some of them out to see what they do.

- References
  - [You can find the documentation for the network create command](https://docs.docker.com/engine/reference/commandline/network_create/)
  
# Free networking features


First up is Service Discovery. When you create a service, it gets a unique name. That name gets registered with the swarm DNS. And, every service uses the swarm DNS for name resolution. Here is an example for you. We are going to leverage the specifics-over overlay network we created earlier in the creating Docker networks section. We'll create two services (tester1 and tester2) attached to that network, then we will connect to a container in the tester1 services and ping the tester2 service by name. Check it out:

```
ocker network create -d overlay \
> --subnet=172.30.0.0/24 \                                                       > --ip-range=172.30.0.0/28 \
> --gateway=172.30.0.254 \
> specifics-over
0cbhovsunneewllheqgod32jq
[manager1] (local) root@192.168.0.23 ~
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
7e8a0b2f44ad        bridge              bridge              local
21861fa3504d        docker_gwbridge     bridge              local
2a404bf7784f        host                host                local
f9tnnn6q58zy        ingress             overlay             swarm
a3bfef39cbad        none                null                local
0cbhovsunnee        specifics-over      overlay             swarm
[manager1] (local) root@192.168.0.23 ~
$ docker service create --detach --replicas 3 --name tester1 \
> --network specifics-over alpine tail -f /dev/null
igzb7bldsck4rg9qklg6ndilz
[manager1] (local) root@192.168.0.23 ~
$ docker service create --detach --replicas 3 --name tester2 \> --network specifics-over alpine tail -f /dev/null
lt6772clx7okwjndvvijcpmk4
$ docker container exec -it tester1.3.<GET THE NAME> ping -c 3 tester2

```
Note that I typed the first part of the service name (tester1) and used command-line completion by hitting Tab to fill in the container name for the exec command. But, as you can see, I was able to reference the tester2 service by name from within a tester1 container. 

For free!

- The second free feature we get is Load balancing. This powerful feature is pretty easy to understand. It allows traffic intended for a service to be sent to any host in a swarm regardless of whether that host is running a replica of the service. 

- Imagine a scenario where you have a six-node swarm cluster, and a service that has only one replica deployed. You can send traffic to that service via any host in the swarm and know that it will arrive at the service's one container no matter which host the container is actually running on. In fact, you can direct traffic to all hosts in the swarm using a load balancer, say in a round-robin model, and each time traffic is sent to the load balancer, that traffic will get delivered to the app container without fail. 

- Pretty handy, right? Again, for free!

- References
   - [Want to have a go at service discovery? Then check out](https://training.play-with-docker.com/swarm-service-discovery/)
   - [You can read about swarm service load balancing](https://docs.docker.com/engine/swarm/key-concepts/#load-balancing)
   
   
#  Which Docker network driver should I use?

- The short answer to that question is the right one for the job. That means there is no single network driver that is the right fit for every situation. If you're doing work on your laptop, running with swarm inactive, and you just need your containers to be able to communicate with each other, the simple bridge mode driver is ideal. 

If you have multiple nodes and just need container-to-container traffic, the overlay driver is the right one to use. This one works well in AWS, if you are within the container-to-container realm. If you need container-to-VM or container-to-physical-server communication (and can tolerate promiscuous mode), the MACVLAN driver is the way to go. Or, if you have a more complex requirement, one of the many remote drivers might be just what the doctor ordered. 

I've found that for most multi-host scenarios, the overlay driver will get the job done, so I would recommend that you enable swarm mode, and give the overlay driver a try before you ramp up to any of the other multi-host options.



   


    
    
