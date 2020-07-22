---
layout: default
title: ReplicationController 
parent: CKA / CKAD Certification Workshop Track
nav_order: 13
---


# ReplicationController 

Note: A Deployment that configures a ReplicaSet is now the recommended way to set up replication.

- A ReplicationController ensures that a specified number of pod replicas are running at any one time. 
In other words, a ReplicationController makes sure that a pod or a homogeneous set of pods is always up and available.


![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/replica-cotroller.png)


# How a ReplicationController Works?

If there are too many pods, the ReplicationController terminates the extra pods. If there are too few, the ReplicationController starts more pods.
Unlike manually created pods, the pods maintained by a ReplicationController are automatically replaced if they fail, are deleted, or are terminated. 
For example, your pods are re-created on a node after disruptive maintenance such as a kernel upgrade. For this reason, you should use a ReplicationController even if your application 
requires only a single pod. A ReplicationController is similar to a process supervisor, but instead of supervising individual processes on a single node, the ReplicationController 
supervises multiple pods across multiple nodes.

ReplicationController is often abbreviated to “rc” in discussion, and as a shortcut in kubectl commands. A simple case is to create one ReplicationController 
object to reliably run one instance of a Pod indefinitely. A more complex use case is to run several identical replicas of a replicated service, such as web servers.

# ReplicationController manifest file syntax:
```

## ---------------------------------
## ReplicationController YAML syntax
## ---------------------------------

apiVersion: v1
kind: ReplicationController
metadata:
  <replicationcontrollerMetadata>
spec:
  <replicationcontrollerSpec>

```


- replicationcontrollerMetadata attributes:

    - annotations: Key value map stored with a resource for arbitrary metadata.
    - clusterName: The name of the cluster which the object belongs to.
    - creationTimestamp: Timestamp representing the server time when this object was created.
    - deletionGracePeriodSeconds: Seconds allowed for gracefully terminate before forcefully removed from the system.
    - deletionTimestamp: RFC 3339 date and time at which this resource will be deleted. Is not directly settable by a client.
    - finalizers: Must be empty before the object is deleted from the registry.
    - generateName: Optional prefix, used by the server, to generate a unique name ONLY IF the Name field has not been provided.
    - generation: A sequence number representing a specific generation of the desired state. Populated by the system. Read-only.
    - initializers: An initializer is a controller which enforces some system invariant at object creation time.
    - labels: Map of string keys and values that can be used to organize and categorize (scope and select) objects.
    - name: Name must be unique within a namespace.
    - namespace: Namespace defines the space within each name must be unique, default namespace is default.
    - ownerReferences: List of objects depended by this object.
    - resourceVersion: An opaque value that represents the internal version of this object used to determine when objects have changed.
    - selfLink: SelfLink is a URL representing this object. Populated by the system. Read-only.
    - uid: UID is the unique in time and space value for this object
    
- replicationcontrollerSpec attributes:

   - minReadySeconds: Minimum number of seconds for which a newly created pod should be ready without any of its container crashing, for it to be considered available. Defaults to 0
   - replicas: Replicas is the number of desired replicas. This is a pointer to distinguish between explicit zero and unspecified. Defaults to 1.
   - selector: Selector is a label query over pods that should match the Replicas count. If Selector is empty, it is defaulted to the labels present on the Pod template. Label keys and values that must match in order to be controlled by this replication controller, if empty defaulted to labels on Pod template.
   - template: Template is the object that describes the pod that will be created if insufficient replicas are detected.

# Step 1: Create the replication controller.

```


#########################################
## ReplicationController In Kubernetes ##
#########################################
## Prerequisites
## One Kubernetes Cluster with at least one node
## You can install your own or use online versions using below links
## https://www.katacoda.com/courses/kubernetes/playground
## http://labs.play-with-k8s.com/

## -----------------------------
## Create replication controller
## -----------------------------

## Create the replicationcontroller manifest file
mkdir myapp && vi myapp/myreplicationcontrollerv1.yaml
--------------------
apiVersion: v1
kind: ReplicationController
metadata:
  name: myreplicationcontroller
  labels:
    app: myapp
spec:
  replicas: 2
  selector:
    app: myapp
  template:
    metadata:
      labels:
        app: myapp
    spec:
      containers:
      - image: nginx
        name: mycontainer
        ports:
          - containerPort: 80
            name: http
---------------------
:wq

## Validate the yaml configuration file
kubectl create --dry-run --validate -f myapp/myreplicationcontrollerv1.yaml

## Create the replication controller
kubectl apply -f myapp/myreplicationcontrollerv1.yaml


```
# Step 2: View replication controller details.

```
## -----------------------------------
## View replication controller details
## -----------------------------------

## View the replication controller
kubectl get replicationcontroller myreplicationcontroller
kubectl get rc ## abbreviation of replicationcontroller
kubectl get rc -l app=myapp ##list pod by lebel
kubectl get rc -o wide ##get detailed output

## View rc configuration
kubectl get rc myreplicationcontroller -o yaml

## View rc details
kubectl describe rc myreplicationcontroller

## View rc lebels
kubectl get rc --show-labels

## View the pod
kubectl get pod -l app=myapp -o wide

## NAME                            READY   STATUS    RESTARTS   AGE     IP            NODE      NOMINATED NODE   READINESS GATES
## myreplicationcontroller-g7rp4   1/1     Running   0          5m25s   10.244.1.14   system3   <none>           <none>
## myreplicationcontroller-vh79k   1/1     Running   0          5m25s   10.244.2.18   system2   <none>           <none>

```
# Step 3: Scale up and scale down the number of replicas.

```
## ------------------------------------------
## Scale up and scale down number of replicas
## ------------------------------------------

## Scale up number of pods to three
kubectl scale rc myreplicationcontroller --replicas=3

## View pods
kubectl get pod -l app=myapp -o wide

## Scale down number of pods to one
kubectl scale rc myreplicationcontroller --replicas=1

## View pods
kubectl get pod -l app=myapp -o wide

## Check the difference between configuration file and actual state
kubectl diff -f myapp/myreplicationcontrollerv1.yaml

## Note the count of replica is 1 in actual but 2 in config file.

## Edit the rc to make consistent with manifest file
## vi editor will be opened, change replicas: 1 from 2 and save
kubectl edit -f myapp/myreplicationcontrollerv1.yaml

## View pods
kubectl get pod -l app=myapp -o wide


```
Step 4: Delete a pod managed by replication controller.
```
## ---------------------------
## Delete a pod managed by rc
## ---------------------------

## View pods
kubectl get pod

## NAME                            READY   STATUS    RESTARTS   AGE
## myreplicationcontroller-6cfsb   1/1     Running   0          112s
## myreplicationcontroller-g7rp4   1/1     Running   0          20m

## Delete a pod
kubectl delete pod myreplicationcontroller-6cfsb #replace pod name

## Get the pod
kubectl get pod

## NAME                            READY   STATUS    RESTARTS   AGE
## myreplicationcontroller-78rth   1/1     Running   0          10s
## myreplicationcontroller-g7rp4   1/1     Running   0          21m

## One new pod has been autometically spinned up to make the replica count 2


```
# Step 5: Make a connection to the pod managed by replication controller.

```
## -------------------------------------------------
## Connect to your pods under replication controller
## -------------------------------------------------

## Expose your rc
kubectl expose rc myreplicationcontroller --type=NodePort --port=80

## Get pod ip address
kubectl get pod -o wide

## NAME                            READY   STATUS    RESTARTS   AGE     IP            NODE      NOMINATED NODE   READINESS GATES
## myreplicationcontroller-78rth   1/1     Running   0          2m37s   10.244.1.15   system3   <none>           <none>
## myreplicationcontroller-g7rp4   1/1     Running   0          23m     10.244.1.14   system3   <none>           <none>

## Call your pod uisng pod ip address (nginx web server)
curl 10.244.1.15:80
curl 10.244.1.14:80


```

# Step 6: Cleanup.

```
## -------
## Cleanup
## -------

## delete the nodeport service
kubectl delete service myreplicationcontroller

## delete the replication controller
kubectl delete rc myreplicationcontroller

## delete myapp directory
rm -rf myapp



```


