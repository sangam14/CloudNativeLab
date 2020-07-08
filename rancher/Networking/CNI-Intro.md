---
layout: default
title: CNI - Container Networking Interface  
parent: Rancher Networking
nav_order: 10
---

# CNI - Container Networking Interface  


- The Container Networking Interface (CNI) project is also under the governance of the CNCF. It provides a specification and a series of libraries for
writing plugins to configure network interfaces in Linux containers.
- The specification requires that providers implement their plugin as a binary executable that the container engine invokes. Kubernetes does this via the Kubelet process running on each node of the cluster.
- The CNI specification expects the container runtime to create a new network namespace before invoking the CNI plugin. The plugin is then responsible for connecting the container’s network with that of the host. It does this by creating the virtual Ethernet devices that we discussed earlier.


# Kubernetes and CNI
- Kubernetes natively supports the CNI model. It gives its users the freedom to choose the network provider or product best suited for their needs.
- To use the CNI plugin, pass `--network-plugin=cni` to the Kubelet when launching it. If your environment is not using the default configuration directory (`/etc/cni/net.d`),
pass the correct configuration directory as a value to` --cni-conf-dir`. 
- The Kubelet looks for the CNI plugin binary at `/opt/cni/bin`, but you can specify an alternative location with `--cni-bin-dir`.

The CNI plugin provides IP address management for the Pods and builds routes for the virtual interfaces. To do this, the plugin interfaces with an IPAM plugin that is also part of the CNI specification. The IPAM plugin must also be a single executable that the CNI plugin consumes. The role of the IPAM plugin is to provide to the CNI plugin the gateway, IP subnet, and routes for the Pod.
