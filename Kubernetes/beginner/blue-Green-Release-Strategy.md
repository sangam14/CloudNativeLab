---
layout: default
title:  Blue/Green Release Strategy
parent: Kubernetes For Beginner
nav_order: 25
---

# Blue/Green Release Strategy

The Blue/Green deployment involves having two sets of identical hardware. The software application is deployed to both environments at the same time. 
However, only one of the environments receives live traffic while the other remains idle. When a new version of the application is ready, 
it gets deployed to the blue environment. The network is directed to the blue environment through a router or a similar mechanism. If problems 
are detected on the new release and a rollback is needed,
the only action that should be done is redirecting traffic back to the green environment.

- The advantages of this strategy are that - unlike rolling update - there is zero downtime during the deployment process,
although there’s never more than one version of the application running at the same time.

- The drawback, however, is that you need to double the resources hosting the application, which may increase your costs.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/blue-green.png)

the idea is to create a second Deployment with the new application version (blue) while the original one is still running (green).
Once all the Pods in the Blue deployment are ready, we instruct the Service to switch to the Blue deployment (by changing the Pod Selector appropriately).
If a rollback is required, we shift the selector back to the green Pods. Let’s see how this can be done in our example.


First, let’s destroy the current Deployment:

```
kubectl delete deployment mywebserver

```

- lets create three files :- 
  - nginx_deployment_green.yaml
  - nginx_deployment_blue.yaml
  - Nginx_service.yaml


```
//nginx_deployment_green.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mywebserver-green
spec:
  replicas: 4
  strategy:
	type: Recreate
  selector:
	matchLabels:
  	app: nginx_green
  template:
	metadata:
  	labels:
    	app: nginx_green
	spec:
  	containers:
  	- image: YOUR_DOCKER_HUB_USERNAME/mywebserver:1
    	name: nginx


```
- The deployment is using v1 of our application. We also appended “-green” to the Pod tags and their selectors denoting that this is 
the green deployment. Additionally, the deployment name is mywebserver_green, indicating that this is the green deployment.

- Notice that we’re using the Recreate deployment strategy. Using Recreate or RollingUpdate is of no significance here as we are not
relying on the Deployment controller to perform the update.


```
//nginx_deployment_blue.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mywebserver-blue
spec:
  replicas: 4
  strategy:
	type: Recreate
  selector:
	matchLabels:
  	app: nginx_blue
  template:
	metadata:
  	labels:
    	app: nginx_blue
	spec:
  	containers:
  	- image: YOUR_DOCKER_HUB_USERNAME/mywebserver:2
    	name: nginx
```


Our Service should be defined as follows:

```
---
apiVersion: v1
kind: Service
metadata:
  name: mywebservice
spec:
  selector:
	app: nginx_blue
  ports:
	- protocol: TCP
  	port: 80
  	targetPort: 80
  type: NodePort


```

# Testing the Blue/Green Deployment


```
$ kubectl apply -f nginx_deployment_blue.yaml
deployment.apps/mywebserver-blue created

```

And the green one:

```
$ kubectl apply -f nginx_deployment_green.yaml
deployment.apps/mywebserver-green created

```

And finally the service:

```
$ kubectl apply -f nginx_service.yaml
service/mywebservice configured

```

Navigating to http://node_ip:32288 shows that we are using version 2 of our application. If we 
need to quickly rollback to version 1, we just change the Service definition in nginx_service.yaml to look as follows:


```
---
apiVersion: v1
kind: Service
metadata:
  name: mywebservice
spec:
  selector:
	app: nginx_green
  ports:
	- protocol: TCP
  	port: 80
  	targetPort: 80
  type: NodePort

```

Now, refreshing the web page shows that we have reverted to version 1 of our application. There was no downtime during this process.
Additionally, we had only one version of our application running at any particular point in time.
