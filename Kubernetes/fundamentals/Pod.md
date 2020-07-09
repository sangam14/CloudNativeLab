# Pod

A pod is a collection of containers sharing a network and mount namespace and is the basic unit of deployment in Kubernetes.
All containers in a pod are scheduled on the same node.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/akmsymlk8s.png)


# Deploy your first pod 
```
apiVersion : v1
kind : Pod 
metadata :
    name : myapp-pod
    labels:
        app: myapp
        type : front-app 
spec: 
      containers: 
        - name : nginx-containers
          image : nginx
        
```
# Launch or Apply POD


sangam$ kubectl create -f pod.yml 
pod/myapp-pod created

# OR 

kubectl apply Creates and Updates Resources through local or remote files.

sangam$ kubectl apply -f pod.yml
pod/myapp-pod configured

# Check All running Pod And States From Current Namespaces 

```
 sangam$ kubectl get pods
NAME                                 READY   STATUS                       RESTARTS   AGE
myapp-pod                            1/1     Running                      0          6m26s

```

# Check All running Pod And States From All Namespaces 
```
kubectl get pods --all-namespaces 
```
# check describetion details of pod 
```
sangam$ kubectl describe pods myapp-pod 
Name:               myapp-pod
Namespace:          sangam14
Priority:           0
PriorityClassName:  <none>
Node:               gke-us-central1-cloud-okteto-com-pro-a09dced8-a7x1/10.128.0.12
Start Time:         Thu, 09 Jul 2020 10:56:50 +0530
Labels:             app=myapp
                    type=front-app
Annotations:        cni.projectcalico.org/podIP: 10.4.134.166/32
                    container.apparmor.security.beta.kubernetes.io/nginx-containers: runtime/default
                    kubectl.kubernetes.io/last-applied-configuration:
                      {"apiVersion":"v1","kind":"Pod","metadata":{"annotations":{},"labels":{"app":"myapp","type":"front-app"},"name":"myapp-pod","namespace":"s...
                    kubernetes.io/egress-bandwidth: 5M
                    kubernetes.io/ingress-bandwidth: 5M
                    kubernetes.io/limit-ranger:
                      LimitRanger plugin set: cpu, memory request for container nginx-containers; cpu, memory limit for container nginx-containers
                    kubernetes.io/psp: cloud-okteto-enterprise-restrictive
                    seccomp.security.alpha.kubernetes.io/pod: runtime/default
Status:             Running
IP:                 10.4.134.166
Containers:
  nginx-containers:
    Container ID:   docker://2380ea5fcd8b2b71dfb8ea807fd0a150b3269a9b92bd00786a354da3e4a0add9
    Image:          nginx
    Image ID:       docker-pullable://nginx@sha256:0efad4d09a419dc6d574c3c3baacb804a530acd61d5eba72cb1f14e1f5ac0c8f
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 09 Jul 2020 10:56:53 +0530
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  1Gi
    Requests:
      cpu:     0
      memory:  0
    Environment Variables from:
      okteto-secrets  Secret  Optional: true
    Environment:      <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from default-token-j7mjh (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  default-token-j7mjh:
    Type:        Secret (a volume populated by a Secret)
    SecretName:  default-token-j7mjh
    Optional:    false
QoS Class:       Burstable
Node-Selectors:  okteto-node-pool=pro
Tolerations:     node.kubernetes.io/not-ready:NoExecute for 300s
                 node.kubernetes.io/unreachable:NoExecute for 300s
                 okteto-node-pool=pro:NoSchedule
Events:
  Type    Reason     Age   From                                                         Message
  ----    ------     ----  ----                                                         -------
  Normal  Scheduled  15m   default-scheduler                                            Successfully assigned sangam14/myapp-pod to gke-us-central1-cloud-okteto-com-pro-a09dced8-a7x1
  Normal  Pulling    15m   kubelet, gke-us-central1-cloud-okteto-com-pro-a09dced8-a7x1  Pulling image "nginx"
  Normal  Pulled     15m   kubelet, gke-us-central1-cloud-okteto-com-pro-a09dced8-a7x1  Successfully pulled image "nginx"
  Normal  Created    15m   kubelet, gke-us-central1-cloud-okteto-com-pro-a09dced8-a7x1  Created container nginx-containers
  Normal  Started    15m   kubelet, gke-us-central1-cloud-okteto-com-pro-a09dced8-a7x1  Started container nginx-containers

```

# Get The Documentation For Pod Manifests
```

sangam:~ sangam$ kubectl explain pods 
KIND:     Pod
VERSION:  v1

DESCRIPTION:
     Pod is a collection of containers that can run on a host. This resource is
     created by clients and scheduled onto hosts.

FIELDS:
   apiVersion   <string>
     APIVersion defines the versioned schema of this representation of an
     object. Servers should convert recognized schemas to the latest internal
     value, and may reject unrecognized values. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#resources

   kind <string>
     Kind is a string value representing the REST resource this object
     represents. Servers may infer this from the endpoint the client submits
     requests to. Cannot be updated. In CamelCase. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#types-kinds

   metadata     <Object>
     Standard object's metadata. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#metadata

   spec <Object>
     Specification of the desired behavior of the pod. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status

   status       <Object>
     Most recently observed status of the pod. This data may not be up to date.
     Populated by the system. Read-only. More info:
     https://git.k8s.io/community/contributors/devel/api-conventions.md#spec-and-status




```
