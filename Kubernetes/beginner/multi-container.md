---
layout: default
title: Running Multiple Containers in a Single Pod
parent: Kubernetes For Beginner
nav_order: 14
---

# Running Multiple Containers in a Single Pod

## Anatomy of a Pod 

- Pods are designed to run multiple cooperative processes that should act as a cohesive unit. Those processes are wrapped in containers.
- All the containers that form a Pod are running on the same machine. A Pod cannot be split across multiple nodes.
- All the processes (containers) inside a Pod share the same set of resources, and they can communicate with each other throughlocalhost . One of those shared resources is storage.
- A volume (think of it as a directory with shareable data) defined in a Pod can be accessed by all the containers thus allowing them all to share the same data.


We’ll explore storage and volumes in more depth later on. For now, let’s take a look at the `multicontainer-pod.yaml` specification.

```
cat multicontainer-pod

```

The output is as follows.

```
kind: Pod
metadata:
  name: multicontainer-pod
spec:
  containers:
  - name: producer
    image: ubuntu
    command: ["/bin/bash"]
    args: ["-c", "while true; do echo $(hostname) $(date) >> /var/log/index.html; sleep 10; done"]
    volumeMounts:
    - name: webcontent
      mountPath: /var/log
  - name: consumer
    image: nginx
    ports:
      - containerPort: 80
    volumeMounts:
    - name: webcontent
      mountPath: /usr/share/nginx/html
  volumes:
  - name: webcontent 
    emptyDir: {}
    
```

Review the code for a multi-container pod, the volume webcontent is an emptyDir...essentially a temporary file system.
This is mounted in the containers at mountPath, in two different locations inside the container.
As producer writes data, consumer can see it immediatly since it's a shared file system. `more multicontainer-pod.yaml`
 
- Let's create our multi-container Pod.

```
kubectl apply -f multicontainer-pod.yaml
```
 
- Let's connect to our Pod...not specifying a name defaults to the first container in the configuration

```
kubectl exec -it multicontainer-pod -- /bin/sh
ls -la /var/log
tail /var/log/index.html
exit

```
- Let's specify a container name and access the consumer container in our Pod

```
kubectl exec -it multicontainer-pod --container consumer -- /bin/sh
ls -la /usr/share/nginx/html
tail /usr/share/nginx/html/index.html
exit
```
- This application listens on port 80, we'll forward from 8080->80

```
kubectl port-forward multicontainer-pod 8080:80 
curl http://localhost:8080
 ```
 - Kill our port-forward.
 
```
fg
ctrl+c
```
- delete specific pod 

```
kubectl delete pod multicontainer-pod

```

## Formatting the Output 

Let’s say that we want to retrieve the names of the containers in a Pod. The first thing we’d have to do is get familiar 
with Kubernetes API. We can do that by going to Pod v1 core documentation. While reading the documentation will become mandatory sooner 
or later, we’ll use a simpler route and inspect the output from Kubernetes.

```
kubectl get -f multicontainer-pod -o json

```
The get command that would filter the output and retrieve only the names of the containers is as follows.

```
kubectl get -f multicontainer-pod.yaml \
    -o jsonpath="{.spec.containers[*].name}"
    
```

We used jsonpath as the output format and specified that we want to retrieve names of all the containers from the spec . The ability to 
filter and format information might not look that important right now but, once we move into more complex scenarios, it will prove to be invaluable. 
That will become especially evident when we try to automate the processes and requests sent to Kubernetes API.


## Executing Commands Inside the Pod 


How would we execute a command inside the Pod? Unlike the previous examples that did a similar task, this time we have two containers in the Pod,
so we need to be more specific.

```
kubectl exec -it -c  webcontent 

```



