---
layout: default
title: kubernetes Configmap
parent: CKA / CKAD Certification Workshop Track
nav_order: 19
---

# What is a ConfigMap in Kubernetes?

A ConfigMap is a dictionary of configuration settings. This dictionary consists of key-value pairs of strings. Kubernetes provides these values to your containers.
Like with other dictionaries (maps, hashes, ...) the key lets you get and set the configuration value.

# Why would you use a ConfigMap in Kubernetes?

Use a ConfigMap to keep your application code separate from your configuration.
It is an important part of creating a Twelve-Factor Application.
This lets you change easily configuration depending on the environment (development, production, testing) and to dynamically change configuration at runtime. 

# What is a ConfigMap used for?

A ConfigMap stores configuration settings for your code. Store connection strings, public credentials, hostnames, and URLs in your ConfigMap.


# How does a ConfigMap work?
- Here's a quick animation I made showing how a ConfigMap works in Kubernetes.
- First, you have multiple ConfigMaps, one for each environment.
- Second, a ConfigMap is created and added to the Kubernetes cluster.
- Third, containers in the Pod reference the ConfigMap and use its values.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/configmap-diagram.gif)



# Create the environment variables in the text file.

```shell
$ echo -e "DB_URL=localhost:3306\nDB_USERNAME=postgres" > config.txt
```

Create the ConfigMap and point to the text file upon creation.

```shell
$ kubectl create configmap db-config --from-env-file=config.txt
configmap/db-config created
$ kubectl run backend --image=nginx --restart=Never -o yaml --dry-run > pod.yaml
```

The final YAML file should look similar to the following code snippet.

```yaml
apiVersion: v1
kind: Pod
metadata:
  creationTimestamp: null
  labels:
    run: backend
  name: backend
spec:
  containers:
  - image: nginx
    name: backend
    envFrom:
      - configMapRef:
          name: db-config
    resources: {}
  dnsPolicy: ClusterFirst
  restartPolicy: Never
status: {}
```

Create the Pod by pointing the `create` command to the YAML file.

```shell
$ kubectl create -f pod.yaml
```

# Log into the Pod and run the `env` command.

```shell
$ kubectl exec backend -it -- /bin/sh
# env
DB_URL=localhost:3306
DB_USERNAME=postgres
...
# exit
```

## Optional

> How would you approach hot reloading of values defined by a ConfigMap consumed by an application running in Pod?

Changes to environment variables are only reflected if the Pod is restarted. Alternatively, you can mount a ConfigMap as file and poll changes from the mounted file periodically, however, it requires the application to build in the logic.
