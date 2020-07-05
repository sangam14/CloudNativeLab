---
layout: default
title: Pod Networking 
parent: Rancher Networking
nav_order: 6
---

# Pod Networking 

The Pod is the smallest unit in Kubernetes, so it is essential to first understand Kubernetes networking in the context of communication between Pods.
Because a Pod can hold more than one container, we can start with a look at how communication happens between containers in a Pod.
Although Kubernetes can use Docker for the underlying container runtime, its approach to networking differs slightly and imposes some basic principles:
 -  Any Pod can communicate with any other Pod without the use of network address translation (NAT). To facilitate this, Kubernetes assigns each Pod an IP address that is routable within the cluster.
 -  A node can communicate with a Pod without the use of NAT.  
 -  A Pod's awareness of its address is the same as how other resources see the address. The host's address doesn't mask it.These principles give a unique and first-class identity to every Pod in the cluster.
Because of this, the networking model is more straightforward and does not need to include port mapping for the running container workloads.
By keeping the model simple, migrations into a Kubernetes cluster require fewer changes to the container and how it communicates.


# The Pause Container
- A piece of infrastructure that enables many networking features in Kubernetes is known as the pause container. 
- This container runs along side the containers defined in a Pod and is responsible for providing the network namespace that the other containers share. 
It is analogous to joining the network of another container that we described in the User Defined Network section above.
- The pause container was initially designed to act as the init process within a PID namespace shared by all containers in the Pod.
It performed the function of reaping zombie processes when a container died. PID namespace sharing is now disabled by default, so unless it has been explicitly enabled in the kubelet, all containers run their process as PID 1.
