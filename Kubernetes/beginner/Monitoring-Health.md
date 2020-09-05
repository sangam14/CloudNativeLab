---
layout: default
title: Monitoring Health
parent: Kubernetes For Beginner
nav_order: 15
---

# Monitoring Health


# Why to Monitor Health? 

The X Docker image is designed to fail on the first sign of trouble. In
cases like that, there is no need for any health checks. When things go wrong:
  - The main process stops.
  - The container hosting the main process stops as well.
  - Kubernetes restarts the failed container.
However, not all services are designed to fail fast. Even those that are might still benefit 
  from additional health checks. For example, a back-end API can be up and running but, due to a memory leak, serves
  requests much slower than expected. Such a situation might benefit from a health check that would verify whether the service
  responds within, for example, two seconds.

## Kubernetes Probes 
 
 We can exploit Kubernetes liveness and readiness probes for that.
 
 - Liveness Probe 
livenessProbe can be used to confirm whether a container should be running. If the probe fails, Kubernetes will kill the
container and apply restart policy which defaults to Always .

- Instead, we’ll explore livenessProbe . 
Both are defined in the same way so the experience with one of them can be easily applied to the other.

# Understanding the Updated Pod Definition 

```
//liveness-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: liveness
spec:
  containers:
  - name: liveness
    image: ubuntu
    tty: true
    livenessProbe:
      exec:
        command:
        - service
        - nginx
        - status
      initialDelaySeconds: 20
      periodSeconds: 5

```

if you notice in above liveness manifest under the `livenessProbe` we have some command like `service' , `nginx', `status` 

We defined that the action should be exec followed with the command and the it will check all 

- We declared that the first execution of the probe should be delayed by five seconds ( initialDelaySeconds ), that requests 
should timeout after two seconds ( timeoutSeconds ), that the process should be repeated every five seconds ( periodSeconds ), 
and ( failureThreshold ) define how many attempts it must try before giving up . 

## Liveness Probe in Action  

Let’s take a look at the probe in action.

```
kubectl craete -f liveness-pod.yaml
```

We created the Pod with the probe. Now we must wait until the probe fails a few times.
A minute is more than enough. Once we’re done waiting, we can describe the Pod.

```
kubectl describe -f liveness-pod.yaml
```

Please visit [Probe v1](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.19/#probe-v1-core) core if you’d like to learn all the available options.

## Pods Are (Almost) Useless (By Themselves) 

- Pods are fundamental building blocks in Kubernetes. In most cases, you will not create Pods directly. Instead, you’ll use higher level constructs like Controllers.

- Pods are disposable. They are not long lasting services. Even though Kubernetes is doing its best to ensure that the containers in a Pod are (almost)
always up-and-running, the same cannot be said for Pods. If a Pod fails, gets destroyed, or gets evicted from a Node, it will not be rescheduled. At least, 
not without a Controller. Similarly, if a whole node is destroyed, all the Pods on it will cease to exist.
Pods do not heal by themselves. Excluding some special cases, Pods are not meant to be created directly.

Note:- Do not create Pods by themselves. Let one of the controllers create Pods for you.
  
  
