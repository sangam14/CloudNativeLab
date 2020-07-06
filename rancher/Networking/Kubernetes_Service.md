---
layout: default
title: Kubernetes Service
parent: Rancher Networking
nav_order: 7
---

# Kubernetes Service


- Pods are ephemeral. The services that they provide may be critical, but because Kubernetes can terminate Pods at any time, they are unreliable endpoints for 
direct communication. For example, the number of Pods in a ReplicaSet might change as the Deployment scales it up or down to accommodate changes in load on 
the application, and it is unrealistic to expect every client to track these changes while communicating with the Pods. 

- Instead, Kubernetes offers the Service resource, which provides a stable IP address and balances traffic across all of the Pods behind it. This abstraction brings stability and a reliable mechanism for communication between microservices.Services which sit in front of Pods use a selector and labels to find the Pods they manage. All Pods with a label that matches the selector receive traffic through the Service. 
Like a traditional load balancer, the service can expose the Pod functionality at any port, irrespective of the port in use by the Pods themselves.


# KUBE-PROXY

- The kube-proxy daemon that runs on all nodes of the cluster allows the Service to map traffic from one port to another.This component configures the Netfilter rules on all of the nodes according to the Service’s definition in the API server. From Kubernetes 1.9 onward it uses the netlink interface to create IPVS rules. These rules direct traffic to the appropriate Pod.


# ClusterIP

- This type of Service is the default and exists on an IP that is only visible within the cluster. It enables cluster resources to reach one another via a known address while maintaining the security boundaries of the cluster itself. 
For example, a database used by a backend application does not need to be visible outside of the cluster, so using a service of type ClusterIP is appropriate. The backend application would expose an API for interacting with records in the database, and a frontend application or remote clients would consume that API.

# NodePort
- A Service of type NodePort exposes the same port on every node of the cluster. The range of available ports is a cluster-level configuration item, and the Service can either choose one of the ports at random or have one designated in its configuration.
This type of Service automatically creates a ClusterIP Service as its target, and the ClusterIP Service routes traffic to the Pods.External load balancers frequently use NodePort services.
They receive traffic for a specific site or address and forward it to the cluster on that specific port.

# NodePort

- A Service of type NodePort exposes the same port on every node of the cluster. The range of available ports is a cluster-level configuration item, and the Service can either choose one of the ports at random 
or have one designated in its configuration. This type of Service automatically creates a ClusterIP Service as its target, and the ClusterIP Service routes traffic to the Pods.External load balancers frequently 
use NodePort services. They receive traffic for a specific site or address and forward it to the cluster on that specific port.

# LoadBalancer

- When working with a cloud provider for whom support exists within Kubernetes, a Service of type LoadBalancer creates a load balancer in that provider's infrastructure. The exact details of how this happens differ between providers, but all create the load balancer asynchronously and configure it to proxy the request to the corresponding Pods via NodePort and ClusterIP Services that it also creates.
In a later section, we explore Ingress Controllers and how to use them to deliver a load balancing solution for a cluster.

# DNS 
- As we stated above, Pods are ephemeral, and because of this, their IP addresses are not reliable endpoints for communication. Although Services solve this by providing a stable address in front of a group of Pods, consumers of the Service still want to avoid using an IP address. 
Kubernetes solves this by using DNS for service discovery.The default internal domain name for a `cluster is cluster.local`. When you create a Service, it assembles a subdomain of` namespace.svc.cluster.local `(where namespace is the namespace in which the service is running) and sets its name as the hostname. 
For example, if the service was named nginx and ran in the default namespace, consumers of the service would be able to reach it as` nginx.default.svc.cluster.local`. If the service's IP changes, the hostname remains the same. There is no interruption of service.The default DNS provider for Kubernetes is KubeDNS, 
but it’s a pluggable component. Beginning with Kubernetes 1.11 CoreDNS is available as an alternative. In addition to providing the same basic DNS functionality within the cluster, CoreDNS supports a wide range of plugins to activate additional functionality.
