# Getting Started with ReplicaSets


## Understanding ReplicaSets 

Most applications should be scalable and all must be fault tolerant. Pods do not provide those features, ReplicaSets do.

- We learned that Pods are the smallest unit in Kubernetes. We also learned that Pods are not fault tolerant. If a Pod is destroyed, Kubernetes
will do nothing to remedy the problem. That is if Pods are created without Controllers.

- The first Controller we’ll explore is called ReplicaSet. Its primary, and pretty much only function, is to ensure that a specified number of
replicas of a Pod matches the actual state (almost) all the time. That means that ReplicaSets make Pods scalable.

- We can think of ReplicaSets as a self-healing mechanism. As long as elementary conditions are met (e.g., enough memory and CPU), Pods associated
with a ReplicaSet are guaranteed to run. They provide fault- tolerance and high availability.

- If you’re familiar with Replication Controllers, it is worth mentioning that ReplicaSet is the next-generation ReplicationController. The only significant 
difference is that  ReplicaSet has extended support for selectors. Everything else is the same. ReplicationController is considered deprecated, so we’ll focus
only on ReplicaSet.

ReplicaSet’s primary function is to ensure that the specified number of replicas of a service are (almost) always running.

Let’s explore ReplicaSet through examples and see how it works and what it does.


```
// replicaset.yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  name: containerlabs-replicaset
spec:
  replicas: 5
  selector:
    matchLabels:
      tier: frontend
  template:
    metadata:
      labels:
        tier: frontend
    spec:
      containers:
      - name: php-redis
        image: nginx

```

The `apiVersion` , `kind` , and `metadata` fields are mandatory with all Kubernetes objects. ReplicaSet is no exception, i.e., it is also a Kubernetes object

- We specified that the apiVersion is apps/v1 .
- The `kind` is `ReplicaSet` and `metadata` has the name key set to `containerlabs-replicaset` . We could have extended `ReplicaSet` metadata with `labels`.
However, we skipped that part since they would serve only for informational purposes. They do not affect the behavior of the ReplicaSet.
You should be familiar with the above three fields since we already
explored them when we worked with Pods. In addition to them, the spec section is mandatory as well.
- The first field we defined in the spec section is replicas . It sets the desired number of replicas of the Pod. In this case, the ReplicaSet should ensure that 5
Pods should run concurrently. If we did not specify the value of the replicas , it would default to 1 
- The next `spec` section is the `selector `. We use it to select which pods should be included in the ReplicaSet. It does not distinguish between the Pods created 
by a ReplicaSet or some other process. In other words, ReplicaSets and Pods are decoupled. If Pods that match the
selector exist, ReplicaSet will do nothing. If they don’t, it will create as many Pods to match the value of the replicas field.
Not only that ReplicaSet creates the Pods that are missing, but it also monitors the cluster and ensures that the desired number of 
replicas is (almost) always running. In case there are already more running Pods with the matching selector , some will be terminated to match the number set in
replicas .
- The last section of the spec field is the template . It is the only required field in the spec , and it has the same schema as a Pod specification. At a minimum, 
the labels of the `spec.template.metadata.labels` section must match those specified in the `spec.selector.matchLabels` . We can set additional labels that will
serve informational purposes only. ReplicaSet will make sure that the number of replicas matches the number of Pods with the same labels.

# Creating the ReplicaSet 
Let’s create the ReplicaSet and experience its advantages first hand.

```
kubectl create -f replicaset.yaml
```

We got the response that the replicaset was created . We can confirm that by listing all the ReplicaSets in the cluster.

```
kubectl get rs

```

```
kubectl describe -f replicaset.yaml


```

Judging by the events, we can see that ReplicaSet created 5 Pods while
trying to match the desired state with the actual state.
Finally, if you are not yet convinced that the ReplicaSet created the missing Pods, we can list all those running in the cluster and confirm it.

```
kubectl get pods --show-labels

```
To be on the safe side, we used the `--show-labels` argument so that we can verify that the Pods in the cluster match those created by the ReplicaSet.





