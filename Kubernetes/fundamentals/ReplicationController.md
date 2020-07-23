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

Faq : 1. replicationcontrollerMetadata attributes : deletionGracePeriodSeconds 
Is it same as parameter at pod specification by name "terminationGracePeriodSeconds" ? 

yes ! as per my personal knowledge ! 

graceful delete option for pods
on the Kubelet.  When a pod is deleted on the API server, a
grace period is calculated that is based on the
Pod.Spec.TerminationGracePeriodInSeconds, the user's provided grace
period, or a default.  The grace period can only shrink once set.
The value provided by the user (or the default) is set onto metadata
as DeletionGracePeriod.

When the Kubelet sees a pod with DeletionTimestamp set, it uses the
value of ObjectMeta.GracePeriodSeconds as the grace period
sent to Docker.  When updating status, if the pod has DeletionTimestamp
set and all containers are terminated, the Kubelet will update the
status one last time and then invoke Delete(pod, grace: 0) to
clean up the pod immediately.


# 

- Docker containers can be terminated any time, due to an auto-scaling policy, pod or deployment deletion or while rolling out an update. In most of such cases, you will probably want to graceful shutdown your application running inside the container.

- In our case, for example, we do want to wait until all current requests (or jobs processing) have completed, but the actual reasons to graceful shutdown an application may be many, including releasing resources, distributed locks or opened connections.

# How it works

- When a pod should be terminated:

   - A SIGTERM signal is sent to the main process (PID 1) in each container, and a “grace period” countdown starts (defaults to 30 seconds - see below to change it).
   - Upon the receival of the SIGTERM, each container should start a graceful shutdown of the running application and exit.
   - If a container doesn’t terminate within the grace period, a SIGKILL signal will be sent and the container violently terminated.
   
   
   refer: https://kubernetes.io/docs/concepts/workloads/pods/pod/#termination-of-pods
   
# A common pitfall while handling the SIGTERM

Let’s say your Dockerfile ends with a CMD in the shell form:

```
CMD myapp

```

- the shell form runs the command with /bin/sh -c myapp, so the process that will get the SIGTERM is actually /bin/sh and not its child myapp. Depending on the actual shell you’re running, it could or could not pass the signal to its children.

- For example, the shell shipped by default with Alpine Linux does not pass signals to children, while Bash does it. If your shell doesn’t pass signals to children, you’ve a couple of options to ensure the signal will be correctly delivered to the app.

# Option #1: run the CMD in the exec form

You can obviously use the CMD in the exec form. This will run myapp instead of /bin/sh -c myapp, but will not allow you to pass environment variables as arguments.

```

CMD [ "myapp" ]

```

Option #2: run the command with Bash

You can ensure your container includes Bash and run your command through it, in order to support environment variables passed as arguments.

```
CMD [ "/bin/bash", "-c", "myapp --arg=$ENV_VAR" ]

```
# How to change the grace period


The default grace period is 30 seconds. As any default, it could fit or couldn’t fit on your specific use cases. There are two way to change it:

    - In the deployment .yaml file
    - On the command line, when you run kubectl delete
    
    
# Deployment

You can customize the grace period setting terminationGracePeriodSeconds at the pod spec level. For example, the following .yaml shows a simple deployment config with a 60 seconds termination grace period.    

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
    name: test
spec:
    replicas: 1
    template:
        spec:
            containers:
              - name: test
                image: ...
            terminationGracePeriodSeconds: 60

```
# Command line

You can also change the default grace period when you manually delete a resource with kubectl delete command, adding the parameter --grace-period=SECONDS. For example:

```
kubectl delete deployment test --grace-period=60

```
Alternatives

There’re some circumstances where a SIGTERM violently kill the application, vanishing all your efforts to gracefully shutdown it. Nginx, for example, quickly exit on SIGTERM, while you should run /usr/sbin/nginx -s quit to gracefully terminate it.

In such cases, you can use the preStop hook. According to the Kubernetes doc, PreStop works as follow:


```

This hook is called immediately before a container is terminated. No parameters are passed to the handler. This event handler is blocking, and must complete before the call to delete the container is sent to the Docker daemon. The SIGTERM notification sent by Docker is also still sent. A more complete description of termination behavior can be found in Termination of Pods.



```

The preStop hook is configured at container level and allows you to run a custom command before the SIGTERM will be sent (please note that the termination grace period countdown actually starts before invoking the preStop hook and not once the SIGTERM signal will be sent).

The following example, taken from the Kubernetes doc, shows how to configure a preStop command.

```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: nginx
spec:
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
        lifecycle:
          preStop:
            exec:
              # SIGTERM triggers a quick exit; gracefully terminate instead
              command: ["/usr/sbin/nginx","-s","quit"]


```

Faq 2 . I could not follow usecase for finalizers and initializers.

```





```









