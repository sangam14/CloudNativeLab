---
layout: default
title: Creating Services through Declarative Syntax
parent: Kubernetes For Beginner
nav_order: 18
---


# Creating Services through Declarative Syntax


## Looking into the Syntax 

We can accomplish a similar result as the one using` kubectl expose `through `noteport-svc.yml`

```
//noteport-svc.yml
kind: Service
apiVersion: v1
metadata:
  name: mongodb-service
spec:
  type: NodePort
  selector:
    app: mongodb
  ports:
    - port: 28017
      nodePort: 30001 
      protocol: TCP
      name: MongoPort
     
  ```  
  
  - You should be familiar with the meaning of `apiVersion` , `kind` , and `metedata`, so we’ll jump straight into the section. 
 - Since we already explored some of the options through the `kubectl expose` command, the spec should be relatively easy to grasp.
    - The type of the Service is set to NodePort meaning that the ports will be available both within the cluster as well as from outside by 
 sending requests to any of the nodes.
 -   The ports section specifies that the requests should be forwarded to the Pods on port 28017 . The nodePort is new. 
 Instead of letting the service expose a random port, we set it to the explicit value of
30001 . Even though, in most cases, that is not a good practice, I thought it might be a good idea to demonstrate that option as well.
The protocol is set to TCP . The only other alternative would be to use UDP . We could have skipped the protocol altogether since TCP 
is the default value but, sometimes, it is a good idea to leave things as a reminder of an option.


## Creating the Service 

Now that there’s no mystery in the definition, we can proceed and create the
Service.

```
kubectl create -f noteport-svc.yml
kubectl get -f noteport-svc.yml

```

We created the Service and retrieved its information from the API server. The output of the latter command is as follows.

```
NAME          TYPE     CLUSTER-IP EXTERNAL-IP PORT(S)         AGE
nginx-example NodePort 10.0.0.129 <none>     28017:30001/TCP 10m

```
Now that the Service is running (again), we can double-check that it is
working as expected by trying to access mongoDB

```
open "http://$IP:30001"
```
Since we fixed the nodePort to 30001, we did not have to retrieve the Port from the API server. Instead, we used the IP of the
Minikube node and the hard-coded port 30001 to open MongoDB the UI.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/replicaset-service.png)

## Request Forwarding 

- Each Pod has a unique IP that is included in the algorithm used when forwarding requests. Actually, it’s not much of an algorithm. 
Requests will be sent to those Pods randomly. That randomness results in something similar to round-robin load balancing. If the number of Pods does
not change, each will receive an approximately equal number of requests.
- Random requests forwarding should be enough for most use cases. If it’s not, we’d need to resort to a third-party solution. However soon, 
when the newer Kubernetes versions get released, we’ll have an alternative to the iptables solution. We’ll be able to apply different types of 
load balancing algorithms like last connection, destination hashing, newer queue, and so on. Still, the current solution is based on iptables, 
and we’ll stick with it, for now.

