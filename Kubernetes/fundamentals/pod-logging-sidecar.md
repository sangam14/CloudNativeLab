---
layout: default
title:  Lab 4 Pods - Pod Logging Sidecar 
parent: CKA / CKAD Certification Workshop Track
nav_order: 7
---


# Lab 4 Pods : Pod Logging Sidecar 

Log-Shipping Sidecar :- 

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/pod-log-sidecar.png)


# Create ` pod-logging-sidecar.yaml ` file with following Contents:- 
```
---
apiVersion: v1
kind: Pod
metadata:
  name: counter-log-sidecar
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
  - name: counter-log-1
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/1.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  - name: counter-log-2
    image: busybox
    args: [/bin/sh, -c, 'tail -n+1 -f /var/log/2.log']
    volumeMounts:
    - name: varlog
      mountPath: /var/log
  volumes:
  - name: varlog
    emptyDir: {}



```

# ` kubectl Create ' Or ` kubectl Apply '

```
kubectl create -f pod-logging-sidecar.yaml 
pod/counter-log-sidecar created
```

# List All Running Pods 

```
sangam:pods sangam$ kubectl get po
NAME                                 READY   STATUS                       RESTARTS   AGE
counter                              1/1     Running                      0          2d2h
counter-log-sidecar                  3/3     Running                      0          4m21s

```
# kubectl logs

```
sangam:pods sangam$ kubectl exec counter-log-sidecar -c count -it bin/sh
/ # ls
bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # cd var/log
/var/log # ls
1.log  2.log
/var/log # 


```
