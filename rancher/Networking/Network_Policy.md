---
layout: default
title: Network Policy
parent: Rancher Networking
nav_order: 8
---

# Network Policy

- In an enterprise deployment of Kubernetes the cluster often supports multiple projects with different goals. Each of these projects has different workloads, and each of these might require a different security policy.Pods, by default, do not filter incoming traffic. 
There are no firewall rules for inter-Pod communication. Instead, this responsibility falls to the NetworkPolicy resource, which uses a specification to define the network rules applied to a set of Pods

- Pods, by default, do not filter incoming traffic. There are no firewall rules for inter-Pod communication. Instead, this responsibility falls to the NetworkPolicy resource, which uses a specification to define the network rules applied to a set of Pods.

Note:- The network policies are defined in Kubernetes, but the CNI plugins that support network policy implementation do the actual configuration and processing. In a later section, we look at CNI plugins and how they work.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/k8s-net-pod.png)


- The image to the right shows a standard three-tier application with a UI, a backend service, and a database, all deployed within a Kubernetes cluster. 
- Requests to the application arrive at the web Pods, which then initiate a request to the backend Pods for data. The backend Pods process the request and perform CRUD operations against the database Pods.
- If the cluster is not using a network policy, any Pod can talk to any other Pod. Nothing prevents the web Pods from communicating directly with the database Pods. If the security requirements of the cluster dictate a need for clear separation between tiers, a network policy enforces it.

- The policy defined below states that the database Pods can only receive traffic from the Pods with the labels `app=myappand` `role=backend`. It also defines that the backend Pods can only receive traffic from Pods with the labels `app=myapp` and `role=web`.

```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: backend-access-ingress
spec:
  podSelector:
   matchLabels:
    app: myapp
    role: backend
ingress:    
- from:
  - podSelector:
     matchLabels:
      app: myapp
      role: web
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: db-access-ingress
spec:  
 podSelector:
  matchLabels:
    app: myapp
    role: db
ingress:
  - from:
    - podSelector:
      matchLabels:
      app: myapp  
      role: backend


```
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/k8s-Xnet-pod.png)


With this network policy in place, Kubernetes blocks communication between the web and database tiers.
