---
layout: default
title: Custom Bridge Network
parent: Rancher Networking
nav_order: 3
---

# Custom Bridge Network

There is no requirement to use the default bridge on the host; it’s easy to create a new bridge network and attach containers to it. 
This provides better isolation and interoperability between containers, and custom bridge networks have better security and features than the default bridge. <br>
  - All containers in a custom bridge can communicate with the ports of other containers on that bridge.
  This means that you do not need to publish the ports explicitly. It also ensures that the communication between them is secure.
  Imagine an application in which a backend container and a database container need to communicate and where we 
  also want to make sure that no external entity can talk to the database. We do this with a custom bridge network in which only the database container and 
  the backend containers reside. You can explicitly expose the backend API to the rest of the world using port publishing. <br>
  -  The same is true with environment variables - environment variables in a bridge network are shared by all containers on that bridge.
  -  Network configuration options such as MTU can differ between applications. By creating a bridge, you can configure the network to best suit 
  the applications connected to it.

- To create a custom bridge network and two containers that use it, run the following commands:

```
$ docker network create mynetwork
42fab9a1b72e1d6e031298ce6e92487e2a6009f7765b4c489716fd0441e10bbe
$ docker run -it --rm --name=container-a --network=mynetwork busybox /bin/sh
Unable to find image 'busybox:latest' locally
latest: Pulling from library/busybox
91f30d776fb2: Pull complete 
Digest: sha256:9ddee63a712cea977267342e8750ecbc60d3aab25f04ceacfa795e6fce341793
Status: Downloaded newer image for busybox:latest
$  docker run -it --rm --name=container-b --network=mynetwork busybox /bin/sh

```
# Container-Defined Network
A specialized case of custom networking is when a container joins the network of another container. This is similar to how a Pod works in Kubernetes.
The following commands launch two containers that share the same network namespace and thus share the same IP address. 
Services running on one container can talk to services running on the other via the localhost address.
```
$ docker run -it --rm --name=container-a busybox /bin/sh
$ docker run -it --rm --name=container-b --network=container:container-a busybox /bin/sh

```
# No Networking

This mode is useful when the container does not need to communicate with other containers or with the outside world. 
It is not assigned an IP address, and it cannot publish any ports.
```
$ docker run --net=none --name busybox busybox ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
```

# Container-to-Container Communication 

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/Container-to-Container.png)

How do two containers on the same bridge network talk to one another? 
In the above diagram, two containers running on the same host connect via the `docker0 bridge`.
If `172.17.0.6` (on the left-hand side) wants to send a request to `172.17.0.7` (the one on the right-hand side), 
the packets move as follows:
- 1. A packet leaves the container via `eth0` and lands on the corresponding `vethxxx` interface.
- 2. The vethxxx interface connects to the `vethyyy` interface via the `docker0 bridge`.
- 3. The `docker0 bridge` forwards the packet to the `vethyyy interface`.
- 4. The packet moves to the eth0 interface within the destination container.
