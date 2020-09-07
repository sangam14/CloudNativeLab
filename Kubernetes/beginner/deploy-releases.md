---
layout: default
title: Deploying New Releases
parent: Kubernetes For Beginner
nav_order: 21
---

# Deploying New Releases

Just as we are not supposed to create Pods directly but using other controllers like ReplicaSet, we are not supposed to create ReplicaSets either. Kubernetes Deployments will create them for us. If you’re wondering why is this so? You’ll have to wait a little while longer to find out.
First, we’ll create a few Deployments and, once we are familiar with the process and the outcomes, it’ll become obvious why they are better at managing ReplicaSets than we are.

# Looking into the Definition 


```
// deploy.yml
kind: Deployment
apiVersion: extensions/v1beta1
metadata:
  name: nginx-deployment
spec:
  # A deployment's specification really only 
  # has a few useful options
  
  # 1. How many copies of each pod do we want?
  replicas: 3

  # 2. How do want to update the pods?
  strategy: Recreate

  # 3. Which pods are managed by this deployment?
  selector:
    # This must match the labels we set on the pod!
    matchLabels:
      deploy: example
  
  # This template field is a regular pod configuration 
  # nested inside the deployment spec
  template:
    metadata:
      # Set labels on the pod.
      # This is used in the deployment selector.
      labels:
        deploy: example
    spec:
      containers:
        - name: nginx
          image: nginx:1.7.9

```


We will regularly add `--record ` to the kubectl create commands. This allows us to track each change to our resources such as a Deployments

```
kubectl create \
    -f deploy.yml \
    --record
kubectl get -f deploy.yml

```

## Describing the Deployment 

```
kubectl describe -f deploy.yml
```

From the Events section, we can observe that the Deployment created a ReplicaSet. Or, to be more precise, that it scaled it. That is interesting.
It shows that Deployments control ReplicaSets. The Deployment created the ReplicaSet which, in turn, created Pods.
Let’s confirm that by retrieving the list of all the objects.

```
kubectl get all

```

 you might be wondering why we created the Deployment at all. You might think that we’d have the same result if we created a ReplicaSet directly. You’d be right.
So far, from the functional point of view, there is no difference between a ReplicaSet created directly or using a Deployment.

The following figure summarizes the cascading effect of deployments resulting in the creation of pods, containers, and replicaSets.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/Deployement-containerlabs.png)



