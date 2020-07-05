---
layout: default
title: An Introduction to Kubernetes Networking
parent: Rancher Networking
nav_order: 5
---


# An Introduction to Kubernetes Networking

- Kubernetes networking builds on top of the Docker and Netfilter constructs to tie multiple components together into applications.
Kubernetes resources have specific names and capabilities, and we want to understand those before exploring their inner workings. 


# Pods
- The smallest unit of deployment in a Kubernetes cluster is the Pod, 
and all of the constructs related to scheduling and orchestration assist in the deployment and management of Pods.

- In the simplest definition, a Pod encapsulates one or more containers. Containers in the same Pod always run on the same host. 
They share resources such as the network namespace and storage.


- Each Pod has a routable IP address assigned to it, not to the containers running within it. Having a shared network space for all containers
means that the containers inside can communicate with one another over the `localhost` address,a feature not present in traditional Docker networking.

- The most common use of a Pod is to run a single container. Situations where different processes work on the same shared resource, such as content in a storage volume, benefit from having multiple containers in a single Pod. Some projects inject containers into running Pods to deliver a service. 
An example of this is the Istio service mesh, which uses this injected container as a proxy for all communication.

- Because a Pod is the basic unit of deployment, we can map it to a single instance of an application. For example, a three-tier application that runs a user interface (UI), a backend, and a database would model the deployment of the application on Kubernetes with three Pods. 
If one tier of the application needed to scale, the number of Pods in that tier could scale accordingly

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/k8snet.png)


# Workloads 
- Production applications with users run more than one instance of the application. This enables fault tolerance, where if one instance goes down, another
handles the traffic so that users don't experience a disruption to the service. In a traditional model that doesn't use Kubernetes, these types of deployments 
require that an external person or software monitors the application and acts accordingly.
- Kubernetes recognizes that an application might have unique requirements. Does it need to run on every host? Does it need to handle state to avoid data corruption? Can all of its pieces run anywhere, or do they need special scheduling 
consideration? To accommodate those situations where a default structure won't give the best results, Kubernetes provides abstractions for different workload types.

# REPLICASETT
- the ReplicaSet maintains the desired number of copies of a Pod running within the cluster. 
If a Pod or the host on which it's running fails, Kubernetes launches a replacement. In all cases, Kubernetes works to maintain the desired state of the ReplicaSet.

# DEPLOYMENT
- A Deployment manages a ReplicaSet. Although it’s possible to launch a ReplicaSet directly or to use a ReplicationController, 
the use of a Deployment gives more control over the rollout strategies of the Pods that the ReplicaSet controller manages.
By defining the desired states of Pods through a Deployment, users can perform updates to the image running within the containers and maintain 
the ability to perform rollbacks.

# DAEMONSET
- A DaemonSet runs one copy of the Pod on each node in the Kubernetes cluster. This workload model provides the flexibility to run daemon processes such as log management, monitoring, storage providers, or network providers 
that handle Pod networking for the cluster.

# STATEFULSET
- A StatefulSet controller ensures that the Pods it manages have durable storage and persistent identity. StatefulSets are appropriate for situations where Pods have a similar definition but need a unique identity, ordered deployment and scaling, and storage that persists across Pod rescheduling.


