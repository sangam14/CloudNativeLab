---
layout: default
title: Minikube Installation On Mac
parent: CKA / CKAD Certification Workshop Track
nav_order: 2
---



# Requirements

Minikube requires that VT-x/AMD-v virtualization is enabled in BIOS. To check that this is enabled on OSX / macOS run:

    sysctl -a | grep machdep.cpu.features | grep VMX

If there's output, you're good!

# Prerequisites

- kubectl
- docker (for Mac)
- minikube
- virtualbox

```
brew update && brew install kubectl && brew cask install docker minikube virtualbox
```

# Verify

    docker --version                # Docker version 17.09.0-ce, build afdb6d4
    docker-compose --version        # docker-compose version 1.16.1, build 6d1ac21
    docker-machine --version        # docker-machine version 0.12.2, build 9371605
    minikube version                # minikube version: v0.22.3
    kubectl version --client        # Client Version: version.Info{Major:"1", Minor:"8", GitVersion:"v1.8.1", GitCommit:"f38e43b221d08850172a9a4ea785a86a3ffa3b3a", GitTreeState:"clean", BuildDate:"2017-10-12T00:45:05Z", GoVersion:"go1.9.1", Compiler:"gc", Platform:"darwin/amd64"}        


# Download Kubectl

Linux: https://storage.googleapis.com/kubernetes-release/release/v1.6.1/bin/linux/amd64/kubectl

MacOS: https://storage.googleapis.com/kubernetes-release/release/v1.6.1/bin/darwin/amd64/kubectl

Windows: 
https://github.com/eirslett/kubectl-windows/releases/download/v1.6.3/kubectl.exe

# Minikube

- Project URL: https://github.com/kubernetes/minikube

- Latest Release and download instructions: https://github.com/kubernetes/minikube/releases

- VirtualBox: http://www.virtualbox.org

Minikube on windows:

- Download the latest minikube-version.exe

- Rename the file to minikube.exe and put it in C:\minikube

- Open a cmd (search for the app cmd or powershell)

- Run: cd C:\minikube and enter minikube start

# Test your cluster commands

Make sure your cluster is running, you can check with minikube status.

If your cluster is not running, enter minikube start first.

```

Biradars-MacBook-Air:~ sangam$ minikube start
Starting local Kubernetes v1.10.0 cluster...
Starting VM...
Getting VM IP address...
Moving files into cluster...
Setting up certs...
Connecting to cluster...
Setting up kubeconfig...
Starting cluster components...
Kubectl is now configured to use the cluster.
Loading cached images from config file.

```
Great! You now have a running Kubernetes cluster locally. Minikube started a virtual machine for you, and a Kubernetes cluster is now running in that VM


# Check K8 minikube:
```
Biradars-MacBook-Air:~ sangam$ kubectl get nodes
```


# Should output something like:
```
NAME       STATUS   ROLES    AGE   VERSION
minikube   Ready    master   10d   v1.10.0

```

# run first K8-hello-minikube 
```
Biradars-MacBook-Air:~ sangam$ kubectl run hello-minikube --image=gcr.io/google_containers/echoserver:1.4 --port=8080
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/hello-minikube created
```
in this above kubectl is runing hello-minikube and using --image is flag which downloading simple image from google
and --port is exposing on 8080 port 

```
Biradars-MacBook-Air:~ sangam$ kubectl expose deployment hello-minikube --type=NodePort
service/hello-minikube exposed

```
NodePort: Exposes the service on each Node’s IP at a static port (the NodePort). A ClusterIP service, to which the NodePort service will route, is automatically created. You’ll be able to contact the NodePort service, from outside the cluster, by requesting <NodeIP>:<NodePort>
    
```
Biradars-MacBook-Air:~ sangam$ minikube service hello-minikube --url
http://192.168.99.100:30451

<open a browser and go to that url>
 http://192.168.99.100:30451  

```    
# output     
   
```

CLIENT VALUES:
client_address=172.17.0.1
command=GET
real path=/
query=nil
request_version=1.1
request_uri=http://192.168.99.100:8080/

SERVER VALUES:
server_version=nginx: 1.10.0 - lua: 10001

HEADERS RECEIVED:
accept=text/html,application/xhtml+xml,application/xml;q=0.9,*/*;q=0.8
accept-encoding=gzip, deflate
accept-language=en-us
connection=keep-alive
dnt=1
host=192.168.99.100:30451
upgrade-insecure-requests=1
user-agent=Mozilla/5.0 (Macintosh; Intel Mac OS X 10_14_2) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/12.0.2 Safari/605.1.15
BODY:
-no body in request-

```


# You can check in dashboard too..
```

Biradars-MacBook-Air:~ sangam$ minikube dashboard

Opening http://127.0.0.1:49837/api/v1/namespaces/kube-system/services/http:kubernetes-dashboard:/proxy/ in your default browser...
```


# You should now see your pod and your service
```
Biradars-MacBook-Air:~ sangam$ kubectl get all
NAME                              READY   STATUS    RESTARTS   AGE
pod/hello-node-7f5b6bd6b8-f8lj7   1/1     Running   0          9d

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   10d

NAME                         DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/hello-node   1         1         1            1           9d

NAME                                    DESIRED   CURRENT   READY   AGE
replicaset.apps/hello-node-7f5b6bd6b8   1         1         1       9d

```

