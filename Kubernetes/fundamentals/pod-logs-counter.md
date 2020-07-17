---
layout: default
title:  Lab2 - Pod - Arg Instruction
parent: CKA / CKAD Certification Workshop Track
nav_order: 5
---

## Lab2 : Pod - Args Instruction 



## Create ` pod-logging.yml` File With Following Contents:

```

---
    apiVersion: v1
    kind: Pod
    metadata:
      name: counter
    spec:
      containers:
      - name: count
        image: busybox
        args: ['/bin/sh', '-c', 'i=0; while true; do echo "$i: $(date)"; i=$((i+1)); sleep 1;done']
    
// If you supply a command and args, the default Entrypoint and the default Cmd defined in the Docker image are ignored. Your command is run with your args.

```
## use `kubectl create` or `kubectl apply' 
```
kubectl create -f pod-logging.yml 

```

## List All Running Pod 
```
sangam:~ sangam$ kubectl get po
NAME                                 READY   STATUS                       RESTARTS   AGE
counter                              1/1     Running                      0          89s
myapp-pod                            1/1     Running                      0          25h

```
## Run The Args Specification To Get Logs 
```
Syntax : kubectl logs <pod-name> - c <container-name-specified-in-yml> 
```
it will exec args specific command 

```
kubectl logs counter -c count
0: Fri Jul 10 06:39:17 UTC 2020
1: Fri Jul 10 06:39:18 UTC 2020
2: Fri Jul 10 06:39:19 UTC 2020
3: Fri Jul 10 06:39:20 UTC 2020
4: Fri Jul 10 06:39:21 UTC 2020
5: Fri Jul 10 06:39:22 UTC 2020
6: Fri Jul 10 06:39:23 UTC 2020
7: Fri Jul 10 06:39:24 UTC 2020

```
