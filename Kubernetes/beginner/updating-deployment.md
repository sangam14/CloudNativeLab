---
layout: default
title: Updating Deployments
parent: Kubernetes For Beginner
nav_order: 23
---


# Updating Deployments


Updating the nginx Image #

Let’s see what happens when we set a new image to the Pod.
[ref](https://containerlabs.kubedaily.com/Kubernetes/beginner/deploy-releases.html)

```

kubectl create \
    -f deploy.yml \
    ngnix = nginx:1.8.0
    --record
kubectl get -f deploy.yml


```

It’ll take a while until the new image is pulled.

## Describing the Deployment 
Once it’s done, we can describe the Deployment by checking the events it
created.
```

kubectl describe -f deploy.yml

```

## Looking into the Cluster 
To be on the safe side, we might want to retrieve all the objects from the
cluster.

```

kubectl get all 

```

## Exploring Ways to Update Deployment 

## Updating Using Commands 
 
The kubectl set image command is not the only way to update a Deployment. 
We could also have used as well. The kuectl edit command would be as follows.

The command would be as follows.

```
kubectl edit -f deploy.yml

```

Please do NOT execute it. If you do, you’ll need to type :q followed by the enter key to exit.

The above `edit` command is not a good way to update the definition. It is unpractical and undocumented. 
The `kubectl set image` is more useful if we’d like to integrate Deployment updates with one of the CI/CD tools.


## Updating the YAML File 

Another alternative would be to update the YAML file and execute the `kuectl apply` command. While that is a good idea for applications that do not update
frequently, it does not fit well with those that change weekly, daily, or even hourly.

nginx is one of those that might get updated with a new release only a couple of times a year so having an always up-to-date YAML file in your source code
repository is an excellent practice.


## Finishing off 


We used `kubectl set image` just as a way to introduce you to what’s coming
next when we explore frequent deployments without downtime.

A simple update of Pod images is far from what Deployment offers. To see its real power, we should deploy the API. Since it can be scaled to multiple Pods, 
it’ll provide us with a much better playground.











