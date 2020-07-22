---
layout: default
title: ReplicaSet
parent: CKA / CKAD Certification Workshop Track
nav_order: 12
---

# ReplicaSet

- It is one of the Kubernetes controllers used to make sure that we have a specified number of pod replicas running (A controller in Kubernetes is what takes care of tasks to make sure the
desired state of the cluster matches the observed state). Without it, we will have to create multiple manifests for the number of pods we need which is a lot of work to deploy replicas of a single application. 
In previous versions of Kubernetes, it was called “Replication Controller”. The main difference between the two is that ReplicaSets allow us to use something called “Label Selector”.

- “Labels” are key value pair used to specify attributes of objects that are meaningful and useful to users, so keep in mind that It doesn’t change the way the core system works.
“Label Selectors” is used to identify a set of objects in Kubernetes.


# ReplicaSet by example?

Ok now that we understand what a replicaset is, let us create one. In this example, we will deploy our replica.yaml file which will create a simple frontend nginx app with 3 replicas.

```
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: myapp-replicas
  labels:
    app: myapp
    tier: frontend
spec:
  replicas: 3
  selector:
    matchLabels:
      tier: frontend
    matchExpressions:
      - {key: tier, operator: In, values: [frontend]}
  template:
    metadata:
      labels:
        app: myapp
        tier: frontend
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80



```

# First we run kubectl create command to apply our manifest

```
$ kubectl create -f replica.yaml
replicaset.apps "myapp-replicas" created

```

# Next, we make sure it is created

```
$ kubectl get replicaset
NAME                            DESIRED   CURRENT   READY     AGE
myapp-replicas                  3         3         3         15s

```
# We see that there are 3 deployed and 3 ready. We can also just check the pods.

```
$ kubectl get pod
NAME                            READY     STATUS    RESTARTS   AGE
myapp-replicas-67rkp            1/1       Running   0          33s
myapp-replicas-6kfd8            1/1       Running   0          33s
myapp-replicas-s96sg            1/1       Running   0          33s

```

We see all 3 running with 0 restarts which mean our application is not crashing. We can also describe the object which will give us more details about our replicas.

```
$ kubectl describe replicaset myapp-replicas
Name:         myapp-replicas
Namespace:    default
Selector:     tier=frontend,tier in (frontend)
Labels:       app=myapp
              tier=frontend
Annotations:  
Replicas:     3 current / 3 desired
Pods Status:  3 Running / 0 Waiting / 0 Succeeded / 0 Failed
Pod Template:
  Labels:  app=myapp
           tier=frontend
  Containers:
   nginx:
    Image:        nginx
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  
    Mounts:       
  Volumes:        
Events:
  Type    Reason            Age   From                   Message
  ----    ------            ----  ----                   -------
  Normal  SuccessfulCreate  12m   replicaset-controller  Created pod: myapp-replicas-6kfd8
  Normal  SuccessfulCreate  12m   replicaset-controller  Created pod: myapp-replicas-67rkp
  Normal  SuccessfulCreate  12m   replicaset-controller  Created pod: myapp-replicas-s96sg

```
# What if we no longer need 3 replicas, we now only need 1?

- All we have to do is change the replica field to the value we want and k8s will scale it to that number.
For example “replicas: 1”. If we make the change and re apply, we will only have one replica running.

# Can we remove pod from a ReplicaSets?

- Yes we can. It is as simple as removing the label from the pod and it will be removed from the Set.

# How to cleanup?

- To delete replicaset, all we have to do is run the “kubectl delete replicaset myapp-replicas” this command will delete the replicasets and the pods.

- In general Kubernetes recommend that we use deployment (a higher-level concept that manages ReplicaSets and provides declarative updates to pods along with a lot of other useful features) controller instead of ReplicaSets.
