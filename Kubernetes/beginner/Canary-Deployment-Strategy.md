---
layout: default
title: Canary Deployment Strategy
parent: Kubernetes For Beginner
nav_order: 26
---

# Canary Deployment Strategy


Canary Deployment is a popular release strategy that focuses more on “testing the air” before going with the full deployment.

- The name of Canary Deployment Strategy has its origins rooted back to coal miners. When a new mine is discovered, workers used to carry a
cage with some Canary birds. They placed the cage at the mine entrance. If the birds die, that was an indication of toxic Carbon Monoxide gas emission.

- So, what does coal mining have to do with software deployment? While the implementation is different (way to go, Canaries!), the concept remains the same. 
When software is released using a Canary deployment, a small subset of the incoming traffic is directed to the new application version while the majority 
remains routed to the old, stable version.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/canary-dep.png)

- The main advantage of this method is that you get customer feedback quickly on any new features your application offers. If things go 
wrong, you can easily route all the traffic to the stable version. If enough positive feedback is received, you can gradually increase the portion of
traffic going to the new version until it reaches 100%. 

# Canary Testing Using Kubernetes Deployments And Services


- Assuming that we are currently running version 1 of our application and we need to deploy version 2. 
We want to test some metrics like latency, CPU consumption under different load levels. We’re also collecting feedback 
from the users. If everything looks good, we do a full deployment.

- We’re doing this the old-fashioned way for demonstration purposes, yet some tools, such as Istio, can automate this process.

- The first thing we need to do it create two Deployment definition files; one of them uses version 1 of the image, and the other one 
uses version 2. Both Deployments use the same Pod labels and selectors. The files should look something like the following:

```
//nginx_deployment_stable.yaml:

---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mywebserver-stable
spec:
  replicas: 6
  strategy:
	type: Recreate
  selector:
	matchLabels:
  	app: nginx
  template:
	metadata:
  	labels:
    	app: nginx
	spec:
  	containers:
  	- image: YOUR_DOCKER_HUB_USERNAME/mywebserver:1
    	name: nginx

```


nginx_deployment_canary.yaml

```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mywebserver-canary
spec:
  replicas: 2
  strategy:
	type: Recreate
  selector:
	matchLabels:
  	app: nginx
  template:
	metadata:
  	labels:
    	app: nginx
	spec:
  	containers:
  	- image: YOUR_DOCKER_HUB_USERNAME/mywebserver:2
    	name: nginx

```

Both files look identical except for the Deployment name and the Pod image. Notice that we set the number of replicas on 
the “stable” deployment to 6 while we’re deploying only 2 Pods on the Canary one. This is intentional; 
we need 25% only of our application to serve version 2 while the remaining 75% continues to serve version 1.


# Testing the Canary deployment


```
$ kubectl apply -f nginx_deployment_stable.yaml                                                                                                                                      
deployment.apps/mywebserver-stable created
$ kubectl apply -f nginx_deployment_canary.yaml                                                                                                                                      
deployment.apps/mywebserver-canary created
$ kubectl apply -f nginx_service.yaml                                                                                                                                                
service/mywebservice configured

```

If you refresh the web page http://node_port:32288 several times, you may occasionally see version 2 of the application shows.


- Increasing the percentage of users going to version 2 is as simple as increasing the replicas count on the Canary deployment 
and decreasing the replicas count on the stable one. If you need to rollback, you just set the number of replicas to be 8 (100%) on the
stable Deployment and deleting the Canary Deployment. Alternatively, you can go ahead with the Canary Deployment be reversing this operation.

