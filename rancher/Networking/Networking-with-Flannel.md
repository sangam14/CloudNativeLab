---
layout: default
title: Networking with Flannel
parent: Rancher Networking
nav_order: 10
---

# Networking with Flannel


- Flannel is one of the most straightforward network providers for Kubernetes. It operates at Layer 3 and offloads the actual packet forwarding to a backend such 
as VxLAN or IPSec. It assigns a large network to all hosts in the cluster and then assigns a portion of that network to each host. Routing between containers on
a host happens via the usual channels, and Flannel handles routing between hosts using one of its available options.

- Flannel uses etcd to store the map of what network is assigned to which host. The target can be an external deployment of etcd or the one that Kubernetes itself uses.

- Flannel does not provide an implementation of the NetworkPolicy resource. 

# Running Flannel With Kubenetes

- Flannel Pods roll out as a DaemonSet, with one Pod assigned to each host. To deploy it within Kubernetes, use the `kube-flannel.yaml` manifest from the Flannel repository on Github.

```

kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml

```

- Once Flannel is running, it is not possible to change the network address space or the backend communication format without cluster downtime.

```
kubectl get pods --all-namespacesNAMESPACE     NAME                                      READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-7lfpd                  1/1     Running   0          8m28s
kube-system   coredns-66bff467f8-mx4tq                  1/1     Running   0          8m28s
kube-system   etcd-master                               1/1     Running   0          8m37s
kube-system   katacoda-cloud-provider-58f89f7d9-lcghg   1/1     Running   5          8m28s
kube-system   kube-apiserver-master                     1/1     Running   0          8m37s
kube-system   kube-controller-manager-master            1/1     Running   0          8m37skube-system   kube-flannel-ds-amd64-brdvt               1/1     Running   1          8m20s
kube-system   kube-flannel-ds-amd64-ldt8g               1/1     Running   0          8m28skube-system   kube-keepalived-vip-tr9nf                 1/1     Running   0          8m10s
kube-system   kube-proxy-7gk5t                          1/1     Running   0          8m28s
kube-system   kube-proxy-dqn7c                          1/1     Running   0          8m20s
kube-system   kube-scheduler-master                     1/1     Running   0          8m37s

```


| Network Type 	| Backend 	| Key features                                                                       	|
|--------------	|---------	|------------------------------------------------------------------------------------	|
| Overlay      	| VxLAN   	| - Fast, but with no interhost encryption<br>- Suitable for private/secure networks 	|
| Overlay      	| IPSec   	| - Encrypts traffic between hosts<br>- Suitable when traffic traverses the Internet 	|
| Non Overlay  	| Host-gw 	| - Good performance<br>- Cloud agnostic                                             	|
| Non Overlay  	| AWS VPC 	| - Good performance<br>- Limited to Amazon’s cloud                                  	|

# Flannel Backends 

- VxLAN
   - VxLAN is the simplest of the officially supported backends for Flannel. Encapsulation happens within the kernel, so there is no additional overhead caused by moving data between the kernel and user space
   
   - The VxLAN backend creates a Flannel interface on every host. When a container on one node wishes to send traffic to a different node, the packet goes from the container to the bridge interface in the host’s network namespace. From there the bridge forwards it to the Flannel interface because the kernel route table designates that this interface is the target for the non-local portion of the overlay network. The Flannel network driver wraps the packet in a UDP packet and sends it to the target host.
   
   - Once it arrives at its destination, the process flows in reverse, with the Flannel driver on the destination host unwrapping the packet, sending it to the bridge interface, and from there the packet finds its way into the overlay network and to the destination Pod.
   
-  Host-gw
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/host-gw.png)

- The Host-gw backend provides better performance than VxLAN but requires Layer 2 connectivity between hosts. It operates by creating IP routes to subnets via remote machine addresses
    
- Unlike VxLAN, no Flannel interface is created when using this backend. Instead, each node sends traffic directly to the destination node where the remote network is located.
    
- This backend may require additional network configuration if used in a cloud provider where inter-host communication uses virtual switches.

- UDP
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/UDP.png)
    - The UDP backend is insecure and should only be used for debugging or if the kernel does not support VxLAN.  
    
    
