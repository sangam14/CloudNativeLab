---
layout: default
title:  Lab 3 Pod - Logging Volume 
parent: CKA / CKAD Certification Workshop Track
nav_order: 6
---


# Lab 3 - Pod : Logging Volume 

- You can use a sidecar container in one of the following ways:

    - The sidecar container streams application logs to its own stdout.
    - The sidecar container runs a logging agent, which is configured to pick up logs from an application container.

A pod runs a single container, and the container writes to two different log files, using two different formats. Here's a configuration file for the Pod:

# Create `pod-logging-volum.yml` File With Following Contents 

```
---
apiVersion: v1
kind: Pod
metadata:
  name: counter-log-vol
spec:
  containers:
  - name: count
    image: busybox
    args:
    - /bin/sh
    - -c
    - >
        i=0;
        while true;
        do
          echo "$i: $(date)" >> /var/log/1.log;
          echo "$(date) INFO $i" >> /var/log/2.log;
          i=$((i+1));
          sleep 1;
        done
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}

```
# Use `kubectl create` OR ` kubectl apply` 

```

sangam:~ sangam$ kubectl create -f pod-logging-volum.yml 
pod/counter-log-vol created

```
# List Of All Runnning Pods 

```

sangam:~ sangam$ kubectl get po
NAME                                 READY   STATUS                       RESTARTS   AGE
counter                              1/1     Running                      0          3h39m
counter-log-vol                      1/1     Running                      0          12m
myapp-pod                            1/1     Running                      0          28h

```



# emptyDir

An emptyDir volume is first created when a Pod is assigned to a Node, and exists as long as that Pod is running on that node.
As the name says, it is initially empty. Containers in the Pod can all read and write the same files in the emptyDir volume, though that volume can be mounted 
at the same or different paths in each Container.
When a Pod is removed from a node for any reason, the data in the emptyDir is deleted forever.


# Check Volume from the inside pod 

```
 kubectl exec -it  counter-log-vol -- bin/sh
 cd /var/log
 ls
 cat 1.log 2.log 

```
