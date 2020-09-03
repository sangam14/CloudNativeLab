---
layout: default
title: Getting Started with Pods
parent: Kubernetes For Beginner
nav_order: 9
---

# Getting Started with Pods

## Understanding Pods 

Pods are equivalent to bricks we use to build houses. Both are uneventful and not much by themselves. Yet, they 
are fundamental building blocks without which we could not construct the solution we are set to build.
If you have used Docker or Docker Swarm, you’re probably used to thinking that a container is the smallest unit
and that more complex patterns are built on top of it. With Kubernetes, the smallest unit is a Pod.

## A Pod is a way to represent a running process in a cluster.

- From the Kubernetes’ perspective, there’s nothing smaller than a Pod.
- A Pod encapsulates one or more containers. It provides a unique network IP, attaches storage resources, and also decides how containers should run. Everything in a Pod is tightly coupled.
- We should clarify that containers in a Pod are not necessarily made by Docker. Other container runtimes are supported as well. Still, at the time of this writing, Docker is the 
most commonly used container runtime, and all our examples will use it.
- Since we cannot create Pods without a Kubernetes cluster, our first order of business is to create one.

## Creating A Cluster(minikube)
We’ll create a local Kubernetes cluster using Minikube.

```
minikube start --vm-driver=virtualbox
kubectl get nodes

```
## other Alternative free kubernetes cluster 

- [Okteto](www.okteto.com)
- [play with Kubernetes](https://labs.play-with-docker.com/)

hope everything setup well at your end now ! we will run our first Pod >>
