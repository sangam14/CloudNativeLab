---
layout: default
title: Sequential Breakdown of the Process
parent: Kubernetes For Beginner
nav_order: 17
---

# Sequential Breakdown of the Process

The sequence of events that transpired with the `kubectl create -f replicaset.yaml `
command is as follows.

- 1. Kubernetes client (`kubectl`) sent a request to the API server requesting 
the creation of a ReplicaSet defined in the `replicaset.yaml` file .
- 2. The controller is watching the API server for new events, and it detected that there is a new ReplicaSet object.
- 3. The controller creates 5 new pod definitions because we have configured replica value as 5 in file.
- 4. Since the scheduler is watching the API server for new events, it detected that there are two unassigned Pods.
- 5. The scheduler decided which node to assign the Pod and sent that information to the API server.
- 6. Kubelet is also watching the API server. It detected that the two Pods were assigned to the node it is running on.
- 7. Kubelet sent requests to Docker requesting the creation of the containers that form the Pod.
- 8. Finally, Kubelet sent a request to the API server notifying it that the Pods

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/replicaset-controller-sequence.png)

The sequence we described is useful when we want to understand everything that happened in the cluster from the moment we requested the creation of a new ReplicaSet. However, it might be too confusing so we’ll try 
to explain the same process through a diagram that more closely represents the cluster.


