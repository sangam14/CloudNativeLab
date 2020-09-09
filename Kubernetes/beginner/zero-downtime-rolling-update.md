#  Zero Downtime with Rolling Updates


We create three Dockerfiles as follows:

## Dockerfile_1:

```
//DockerFile.v1
FROM nginx:latest
COPY v1.html /usr/share/nginx/html/index.html

```

## Dockerfile_2:

```
//DockerFile.v2
FROM nginx:latest
COPY v2.html /usr/share/nginx/html/index.html

```

## Dockerfile_3:

```
//DockerFile.v3
FROM nginx:latest
COPY v3.html /usr/share/nginx/html/index.html

```

## The HTML files are as follows:

v1.html

```

<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="X-UA-Compatible" content="ie=edge">
	<title>Release 1</title>
</head>
<body>
	<h1>This is release #1 of the application</h1>
</body>
</html>


```
v2.html
```

<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="X-UA-Compatible" content="ie=edge">
	<title>Release 2</title>
</head>
<body>
	<h1>This is release #2 of the application</h1>
</body>
</html>


```

v3.html
```

<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<meta name="viewport" content="width=device-width, initial-scale=1.0">
	<meta http-equiv="X-UA-Compatible" content="ie=edge">
	<title>Release 2</title>
</head>
<body>
	<h1>This is release #3 of the application</h1>
</body>
</html>

```
Finally, we need to build and push those images:

```
docker build -t sangam14/mywebserver:1 -f Dockerfile_1 .
docker build -t sangam14/mywebserver:2 -f Dockerfile_2 .
docker build -t sangam14/mywebserver:3 -f Dockerfile_3 .
docker push YOUR_DOCKER_HUB_USERNAME/mywebserver:1
docker push YOUR_DOCKER_HUB_USERNAME/mywebserver:2
docker push YOUR_DOCKER_HUB_USERNAME/mywebserver:3

```

# Zero Downtime with Rolling Updates

Let’s create a Deployment for running v1 for our web server. Create a YAML file called `nginx_deployment.yaml` and add the following:

```
---
apiVersion: v1
kind: Service
metadata:
  name: mywebservice
spec:
  selector:
	app: nginx
  ports:
	- protocol: TCP
  	port: 80
  	targetPort: 80
  type: NodePort
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: mywebserver
spec:
  replicas: 4
  strategy:
	type: RollingUpdate
	rollingUpdate:
  	maxSurge: 1
  	maxUnavailable: 1
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
    	readinessProbe:
      	httpGet:
        	path: /
        	port: 80
        	httpHeaders:
        	- name: Host
          	value: K8sProbe


```
Let’s deploy the Service and the Deployment to see that in action:

```
kubectl apply -f nginx_deployment.yaml

```

Now, let’s ensure that our Pods are in the running state:

```
$ kubectl get pods
NAME                       	READY   STATUS	RESTARTS   AGE
mywebserver-68cd66868f-78jgt   1/1 	Running   0      	5m29s
mywebserver-68cd66868f-kdxx9   1/1 	Running   0      	29m
mywebserver-68cd66868f-lh6wz   1/1 	Running   0      	29m
mywebserver-68cd66868f-vvqrh   1/1 	Running   0      	5m29s

```
If we want to actually see the contents of the web page nginx is serving, we need to know the port that the Service is listening at:

```
$ kubectl get svc
NAME       	TYPE    	CLUSTER-IP 	EXTERNAL-IP   PORT(S)    	AGE
kubernetes 	ClusterIP   10.96.0.1  	    	443/TCP    	2d10h
mywebservice   NodePort	10.107.1.198       	80:32288/TCP   32m

```

Our mywebservice Service is using port 32288 to route 
traffic to the Pod on port 80. If you navigate to http://node_ip:3288 

# Upgrading Your Deployments

There is more than one way to update a running Deployment, one of them is modifying the definition file to reflect the new changes and applying it using kubectl
Change the `.spec.template.spec.containers[].image` in the definition file to look as follows:


```
spec:
  	containers:
  	- image: YOUR_DOCKER_HUB_USERNAME/mywebserver:2

```

Clearly, the only thing that changed is the image tag: we need to deploy the second version of our application. Apply the file using kubectl:

```
$ kubectl apply -f nginx_deployment.yaml
service/mywebservice unchanged
deployment.apps/mywebserver configured

```

# Testing the Deployment strategy

```

$ kubectl get pods                                                                                                                                                                    
NAME                       	READY   STATUS          	RESTARTS   AGE
mywebserver-68cd66868f-7w4fc   1/1 	Terminating     	0      	83s
mywebserver-68cd66868f-dwknx   1/1 	Running         	0      	94s
mywebserver-68cd66868f-mv9dg   1/1 	Terminating     	0      	94s
mywebserver-68cd66868f-rpr5f   0/1 	Terminating     	0      	84s
mywebserver-77d979dbfb-qt58n   1/1 	Running         	0      	4s
mywebserver-77d979dbfb-sb9s5   1/1 	Running         	0      	4s
mywebserver-77d979dbfb-wxqfj   0/1 	ContainerCreating   0      	0s
mywebserver-77d979dbfb-ztpc8   0/1 	ContainerCreating   0      	0s
$ kubectl get pods                                                                                                                                                                    
NAME                       	READY   STATUS    	RESTARTS   AGE
mywebserver-68cd66868f-dwknx   0/1 	Terminating   0      	100s
mywebserver-77d979dbfb-qt58n   1/1 	Running   	0      	10s
mywebserver-77d979dbfb-sb9s5   1/1 	Running   	0      	10s
mywebserver-77d979dbfb-wxqfj   1/1 	Running   	0      	6s
mywebserver-77d979dbfb-ztpc8   0/1 	Running   	0      	6s
$ kubectl get pods                                                                                                                                                                    
NAME                       	READY   STATUS	RESTARTS   AGE
mywebserver-77d979dbfb-qt58n   1/1 	Running   0      	25s
mywebserver-77d979dbfb-sb9s5   1/1 	Running   0      	25s
mywebserver-77d979dbfb-wxqfj   1/1 	Running   0      	21s
mywebserver-77d979dbfb-ztpc8   1/1 	Running   0      	21s

```

