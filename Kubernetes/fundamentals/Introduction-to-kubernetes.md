---
layout: default
title: Looking at Kubernetes from the top of a mountain
parent: CKA / CKAD Certification Workshop Track
nav_order: 1
---

# Looking at Kubernetes from the top of a mountain

- Kubernetes is a software system that allows you to easily deploy and manage containerized applications on top of it. It relies on the features of Linux containers to run heterogeneous applications without having to know any internal details of these applications and without having to manually deploy these applications on each host. Because these apps run in containers, they don’t affect other apps running on the same server, which is critical when you run applications for completely different organizations on the same hardware. This is of paramount importance for cloud provid- ers, because they strive for the best possible utilization of their hardware while still having to maintain complete isolation of hosted applications.

- Kubernetes enables you to run your software applications on thousands of computer nodes as if all those nodes were a single, enormous computer. It abstracts away the underlying infrastructure and, by doing so, simplifies development, deployment, and management for both development and the operations teams.
Deploying applications through Kubernetes is always the same, whether your cluster contains only a couple of nodes or thousands of them. The size of the cluster makes no difference at all. Additional cluster nodes simply represent an additional amount of resources available to deployed apps.

 What is Master Node in Kubernetes Architecture?
 {: .label .label-blue }

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/k8s_arch_new.png)

- The Kubernetes Master (Master Node) receives input from a CLI (Command-Line Interface) or UI (User Interface) via an API. These are the commands you provide to Kubernetes.
- You define pods, replica sets, and services that you want Kubernetes to maintain. For example, which container image to use, which ports to expose, and how many pod replicas to run.
- You also provide the parameters of the desired state for the application(s) running in that cluster.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/master-node-k8s.png)
    - Masters: The master is the heart of Kubernetes; it controls and schedules all of the activities in the cluster <br>
    - Nodes: Nodes are the workers that run our containers

 MASTER NODE
 {: .label .label-blue }
- A Kubernetes cluster also contains one or more master nodes that run the Kubernetes control plane. The control plane consists of different processes, such as an API server (provides JSON over HTTP API), scheduler (selects nodes to run containers), controller manager (runs controllers, see below), and etcd (a globally available configuration store).
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/kubernetes-master-node.png)

NODE
{: .label .label-blue }
- A Kubernetes cluster consists of one or more nodes managed by Kubernetes. The nodes are bare-metal servers, on-premises VMs, or VMs on a cloud provider. Every node contains a container runtime (for example, Docker Engine), kubelet (responsible for starting, stopping, and managing individual containers by requests from the Kubernetes control plane), and kube-proxy (responsible for networking and load balancing).
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/worker-node.png)

DASHBOARD AND CLI
{: .label .label-blue }
- A Kubernetes cluster can be managed via the Kubernetes Dashboard, a web UI running on the master node. The cluster can also be managed via the command line tool kubectl, which can be installed on any machine able to access the API server, running on the master node. This tool can be used to manage several Kubernetes clusters by specifying a context defined in a configuration file.

# KUBERNETES BUILDING BLOCKS
- Kubernetes provides basic mechanisms for the deployment, maintenance, and scaling of containerized applications. It uses declarative primitives, or building blocks, to maintain the state requested by the user, implementing the transition from the current observable state to the requested state.

POD
{: .label .label-blue }
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/pods-k8s.png)
- A pod is the smallest deployable unit that can be managed by Kubernetes. A pod is a logical group of one or more containers that share the same IP address and port space. The main purpose of a pod is to support co-located processes, such as an application server and its local cache. Containers within a pod can find each other via localhost, and can also communicate with each other using standard inter-process communications like SystemV semaphores or POSIX shared memory. In other words, a pod represents a “logical host”. Pods are not durable; they will not survive scheduling failures or node failures. If a node where the pod is running dies, the pod is deleted. It can then be replaced by an identical pod, with even the same name, but with a new unique identifier (UID).

LABEL
{: .label .label-blue }
- A label is a key/value pair that is attached to Kubernetes resource, for example, a pod. Labels can be attached to resources at creation time, as well as added and modified at any later time.

CONTROLLER
{: .label .label-blue }
- A controller manages a set of pods and ensures that the cluster is in the specified state. Unlike manually created pods, the pods maintained by a replication controller are automatically replaced if they fail, get deleted, or are terminated. There are several controller types, such as replication controllers or deployment controllers.

REPLICATION CONTROLLER
{: .label .label-blue }
- A replication controller is responsible for running the specified number of pod copies (replicas) across the cluster.

DEPLOYMENT CONTROLLER
 {: .label .label-blue }
- A deployment defines a desired state for logical group of pods and replica sets. It creates new resources or replaces the existing resources, if necessary. A deployment can be updated, rolled out, or rolled back. A practical use case for a deployment is to bring up a replica set and pods, then update the deployment to re-create the pods (for example, to use a new image). Later, the deployment can be rolled back to an earlier revision if the current deployment is not stable.

 REPLICA SET
 {: .label .label-blue }
- A replica set is the next-generation replication controller. A replication controller supports only equality-based selectors, while a replica set supports set-based selectors.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/Deploy-po-container.png)

 SERVICE
 {: .label .label-blue }
- A service uses a selector to define a logical group of pods and defines a policy to access such logical groups. Because pods are not durable, the actual pods that are running may change. A client that uses one or more containers within a pod should not need to be aware of which specific pod it works with, especially if there are several pods (replicas).
- There are several types of services in Kubernetes, including ClusterIP, NodePort, LoadBalancer. A ClusterIP service exposes pods to connections from inside the cluster. A NodePort service exposes pods to external traffic by forwarding traffic from a port on each node of the cluster to the container port. A LoadBalancer service also exposes pods to external traffic, as NodePort service does, however it also provides a load balancer.


