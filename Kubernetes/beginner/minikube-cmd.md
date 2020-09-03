---
layout: default
title: Exploring Minikube Commands
parent: Kubernetes For Beginner
nav_order: 8
---

# Exploring Minikube Commands

## Checking the Environment Variables 

Another useful Minikube command to output the environment variables is `docker-env` :

```
minikube docker-env

```

If you have worked with Docker Machine, you’ll notice that the output is the same. Both docker-machine env and minikube docker-env serve the same
purpose They output the environment variables required for a local Docker client to communicate with a remote Docker server. 
In this case, that Docker server is the one inside a VM created by Minikube.

We assume that you already have Docker installed on your machine. If that’s not the case, 
please go to the [Install Docker](https://docs.docker.com/engine/install/) page and follow the instructions for your operating system.

## Configuring the Shell 

Once Docker is installed, we can connect the client running on your laptop
with the server in the Minikube VM.

```
eval $(minikube docker-env)

```

We evaluated (created) the environment variables provided through the minikube docker-env command. As a result, every command we send to our
local Docker client will be executed on the Minikube VM.

📝 The above command will not result in any output to the console.

## Verification 
We can test that easily by, for example, listing all the running containers on that VM.

```
docker container ls

```

- The containers listed in the output are those required by Kubernetes. We can, in a way, consider them system containers. We won’t discuss each of them. 
As a matter of fact, we won’t discuss any of them. At least, not right away. All you need to know, at this point, is that they make Kubernetes work.

- Since almost everything in that VM is a container, pointing the local Docker client to the service inside, it should be all you need (besides kubectl ). 
Still, in some cases, you might want to SSH into the VM.

```
minikube ssh
docker container ls
exit

```
- We entered into the Minikube VM, listed containers, and got out. There’s no reason to do anything else beyond showing that SSH is possible,
even though you probably won’t use it.

- What else is there to verify? We can, for example, confirm that kubectl is also pointing to the Minikube VM.

```
kubectl config current-context
```

- The output should be a single word, minikube , indicating that kubectl is configured to talk to Kubernetes inside the newly created cluster.
- As an additional verification, we can list all the nodes of the cluster.

```
kubectl get nodes

```
The output is as follows.

```
NAME     STATUS ROLES  AGE VERSION
minikube Ready  master 31m v1.14.0
```

- It should come as no surprise that there is only one node, conveniently called minikube .

- If you are experienced with Docker Machine or Vagrant, you probably noticed a similar pattern. Minikube commands are almost exactly the same as those from 
Docker Machine which, on the other hand, are similar to those from Vagrant.

Let’s make a sneak peek into the components currently running in our tiny cluster.

```
kubectl get all --all-namespaces

```
Behold, the cluster in all its glory. It’s made out of many building blocks we are yet to explore. Moreover, those are only the beginning. 
We’ll be adding more as our needs and knowledge increase. For now, remember that there are many moving pieces. We won’t go into details just yet. 
That would be too much to start with.

## Common Minikube Commands 

Going back to minikube, we can do all the common things we would expect
from a virtual machine.

### Stopping Minikube 

The minikube stop command can be used to stop your cluster. 

This` minikube stop` command shuts down the Minikube Virtual Machine, but preserves all cluster state and data. 
Starting the cluster again will restore it to its previous state

```
minikube stop

```
###  Starting Minikube 
We can start it again

```
minikube start
```

### Deleting Minikube 

The minikube delete command can be used to delete the cluster. This command shuts down and deletes the Minikube Virtual Machine. No data or state is preserved.

```
minikube delete

```
## Specifying the Kubernetes Version 
One interesting feature is the ability to specify which Kubernetes version we’d like to use.

Since Kubernetes is still a young project, we can expect quite a lot of changes at a rapid pace. That will often mean that our production cluster might not be
running the latest version. On the other hand, we should strive to have our local environment as close to production as possible (within reason).
We can create a new cluster based on, let’s say, Kubernetes v1.19.0.

```
minikube start \
    --vm-driver=virtualbox \
    --kubernetes-version="v1.19.0"

```
We created a new cluster. Let’s output versions of the client and the server.

```
We created a new cluster. Let’s output versions of the client and the server.
```



