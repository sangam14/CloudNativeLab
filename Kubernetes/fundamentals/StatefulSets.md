# StatefulSets

StatefulSets make it easier to deploy stateful applications into our Kubernetes cluster.

- StatefulSet is used to manage stateful applications:

   - It manages the deployment and scaling of a set of Pods.
   - It provides guarantees about the ordering and uniqueness of these Pods.
   - However, unlike a Deployment, a StatefulSet maintains a sticky identity for each of their Pods. These pods are created from the same spec, but are not interchangeable: each has a persistent identifier that it maintains across any rescheduling.
   - StatefulSets currently require a Headless Service to be responsible for the network identity of the Pods. We are responsible for creating this Service.
   - StatefulSets do not provide any guarantees on the termination of pods when a StatefulSet is deleted. To achieve ordered and graceful termination of the pods in the StatefulSet, it is possible to scale the StatefulSet down to 0 prior to deletion.
   - The storage for a given Pod must either be provisioned by a PersistentVolume Provisioner based on the requested storage class, or pre-provisioned by an admin.

# Creating a StatefulSet

- StatefulSets are intended to be used with stateful applications and distributed systems. However, in order to demonstrate the basic features of a StatefulSet, we will deploy a simple web application using a StatefulSet.

The following web.yaml creates a Headless Service ("nginx"), to publish the IP addresses of Pods in the StatefulSet, web:

```
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: nginx
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: k8s.gcr.io/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi


```

- A Headless Service, named nginx, is used to control the network domain.
- The Headless service is created by explicitly specifying None for the cluster IP (.spec.clusterIP). So, for headless Services, a cluster IP is not allocated.
- The StatefulSet, named web, has a Spec that indicates that 3 replicas of the nginx container will be launched in unique Pods.
- pod selector: we must set the .spec.selector field of a StatefulSet to match the labels of its .spec.template.metadata.labels.
- The volumeClaimTemplates will provide stable storage using PersistentVolumes provisioned by a PersistentVolume Provisioner. Kubernees creates one 
PersistentVolume for each volumeClaimTemplates. In our case, each Pod will receive a single PersistentVolume with a default StorageClass and 1 Gib of provisioned storage.
- When a Pod is (re)scheduled onto a node, its volumeMounts mount the PersistentVolumes associated with its PersistentVolume Claims. Note that, however, 
the PersistentVolumes associated with the Pods' PersistentVolume Claims are not deleted when the Pods, or StatefulSet are deleted. This must be done manually
- When the StatefulSet Controller creates a Pod, it adds a label, statefulset.kubernetes.io/pod-name, that is set to the name of the Pod. This label allows you
to attach a Service to a specific Pod in the StatefulSet.

We will need to use two terminal windows.

In the first terminal, use kubectl get to watch ('-w') the creation of the StatefulSet's Pods with selector flag ('-l'). 

```

$ kubectl get pods -w -l app=nginx

```

In the second terminal, use kubectl apply to create the Headless Service and StatefulSet defined in the web.yaml:
```
$ kubectl apply -f web.yaml
service/nginx created
statefulset.apps/web created

```
We get the following output from the watch:

```
NAME    READY   STATUS    RESTARTS   AGE
web-0   0/1     Pending   0          0s
web-0   0/1     Pending   0          0s
web-0   0/1     Pending   0          3s
web-0   0/1     ContainerCreating   0          3s
web-0   1/1     Running             0          20s
web-1   0/1     Pending             0          0s
web-1   0/1     Pending             0          0s
web-1   0/1     Pending             0          2s
web-1   0/1     ContainerCreating   0          2s
web-1   1/1     Running             0          4s
web-2   0/1     Pending             0          0s
web-2   0/1     Pending             0          0s
web-2   0/1     Pending             0          2s
web-2   0/1     ContainerCreating   0          2s
web-2   1/1     Running             0          4s


```
Ordered Pod Creation: for a StatefulSet with 3 replicas, when Pods are being deployed, they are created sequentially, 
in order from {0 1 2} as we can see from the output of the kubectl get command in the first terminal. Note that the web-1 Pod is not launched until 
the web-0 Pod is Running and Ready. Same to the web-2 against the web-1.

The command above creates three Pods, each running an nginx webserver. To verify that they were created successfully,
let's get the nginx Service and the web StatefulSet:

```
$ kubectl get service nginx
NAME    TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
nginx   ClusterIP   None         <none>        80/TCP    4m

$ kubectl get statefulset web
NAME   READY   AGE
web    3/3     5m37s


```
# Pods in a StatefulSet

Pods in a StatefulSet have a unique ordinal index and a stable network identity.
Let's examine the Pod's Ordinal Index from the StatefulSet's Pods:

```

$ kubectl get pods -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          22m
web-1   1/1     Running   0          22m
web-2   1/1     Running   0          22m


```

The Pods in a StatefulSet have a sticky, unique identity. This identity is based on a unique ordinal index that is assigned to each Pod by the 
StatefulSet controller. The Pods' names take the form <statefulset name>-<ordinal index>. Since the web StatefulSet has three replicas, it creates three 
Pods, web-0, web-1, and web-2.

Each Pod has a stable hostname based on its ordinal index.

We can use kubectl exec to execute the hostname command in each Pod:

```

$ for i in {0..2}; do kubectl exec web-$i -- sh -c hostname; done
web-0
web-1
web-2

```

We may want to use kubectl run to execute a container (busybox) that provides the nslookup command from the dnsutils package. 
Using nslookup on the Pods' hostnames, we can examine their in-cluster DNS addresses as shown below:

```
$ kubectl run -it --image busybox:1.28 my-dns-test-pod --restart=Never --rm 
If you don't see a command prompt, try pressing enter.

/ # for i in 0 1 2;do nslookup web-$i.nginx;echo;echo '---';done
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-0.nginx
Address 1: 172.17.0.10 web-0.nginx.default.svc.cluster.local

---
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-1.nginx
Address 1: 172.17.0.11 web-1.nginx.default.svc.cluster.local

---
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-2.nginx
Address 1: 172.17.0.12 web-2.nginx.default.svc.cluster.local

---
/ # 



```

While we are making progress, we may want to check the following table, in our case, the first row.
Cluster Domain 

| Cluster Domain 	| Service(ns/name) 	| StatefulSet 	| StatefulSet Domain 	| Pod DNS 	| Pod Hostname 	|
|-	|-	|-	|-	|-	|-	|
| cluster.local 	| default/nginx 	| default/web 	| nginx.default.svc.cluster.local 	| web-0.default.svc.cluster.local ... 	| web-0 ... 	|
| cluster.local 	| foo/nginx 	| foo/web 	| nginx.foo.svc.cluster.local 	| web-0.foo.svc.cluster.local ... 	| web-0 ... 	|
| kube.local 	| foo/nginx 	| foo/web 	| nginx.foo.svc.kube.local 	| web-0.foo.svc.kube.local ... 	| web-0 ... 	|




    - CNAME of the Headless Service : nginx.default.svc.cluster.local (note: "nginx" is the service name - my-svc.my-namespace.svc.cluster.local)
    - SRV records of the Pods : web-0.nginx.default.svc.cluster.local, web-1.nginx.default.svc.cluster.local, and web-2.nginx.default.svc.cluster.local
    
    
```
$ kubectl get pods -o wide
NAME              READY   STATUS    RESTARTS   AGE   IP            NODE       NOMINATED NODE   READINESS GATES
web-0             1/1     Running   0          62m   172.17.0.10   minikube   <none>           <none>
web-1             1/1     Running   0          61m   172.17.0.11   minikube   <none>           <none>
web-2             1/1     Running   0          61m   172.17.0.12   minikube   <none>           <none>

```
The CNAME of the headless service points to SRV records (one for each Pod that is Running and Ready). 
The SRV records point to A record entries that contain the Pods' IP addresses.

# IP addresses associated with the Pods

When the Pods restarted, the Pods' ordinals, hostnames, SRV records, and A record names have not changed, but the IP addresses associated with the Pods may change.
Let's use kubectl delete to delete all the Pods in the StatefulSet, and wait for the StatefulSet to restart them, and for both Pods to transition to Running and Ready:
```
$ kubectl delete pod -l app=nginx
pod "web-0" deleted
pod "web-1" deleted
pod "web-2" deleted

$ kubectl get pod -l app=nginx
NAME    READY   STATUS    RESTARTS   AGE
web-0   1/1     Running   0          22s
web-1   1/1     Running   0          20s
web-2   1/1     Running   0          18s


```

Use kubectl exec and kubectl run to view the Pods hostnames and in-cluster DNS entries:

```
$ for i in 0 1 2; do kubectl exec web-$i -- sh -c 'hostname'; done
web-0
web-1
web-2

$ kubectl run -it --image busybox:1.28 my-dns-test-pod --restart=Never --rm
If you don't see a command prompt, try pressing enter.
/ # for i in 0 1 2;do nslookup web-$i.nginx;echo;echo '---';done
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-0.nginx
Address 1: 172.17.0.2 web-0.nginx.default.svc.cluster.local

---
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-1.nginx
Address 1: 172.17.0.4 web-1.nginx.default.svc.cluster.local

---
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      web-2.nginx
Address 1: 172.17.0.8 web-2.nginx.default.svc.cluster.local

---
/ # 



```

Notice that the IPs of the Pods have changed. This is why it is important not to configure other applications to connect to Pods in a StatefulSet by IP address.

If we need to find and connect to the active members of a StatefulSet, we should query the CNAME of the Headless Service (nginx.default.svc.cluster.local).

From the busybox:

```
/ # nslookup nginx.default.svc.cluster.local
Server:    10.96.0.10
Address 1: 10.96.0.10 kube-dns.kube-system.svc.cluster.local

Name:      nginx.default.svc.cluster.local
Address 1: 172.17.0.8 web-2.nginx.default.svc.cluster.local
Address 2: 172.17.0.4 web-1.nginx.default.svc.cluster.local
Address 3: 172.17.0.2 web-0.nginx.default.svc.cluster.local 

```

The SRV records associated with the CNAME will contain only the Pods in the StatefulSet that are Running and Ready.

If our application already implements connection logic that tests for liveness and readiness, we can use the SRV records of the Pods ( web-0.nginx.default.svc.cluster.local, web-1.nginx.default.svc.cluster.local, and web-2.nginx.default.svc.cluster.local),
as they are stable, and our application will be able to discover the Pods' addresses when they transition to Running and Ready


# Writing to Stable Storage - PersistentVolumeClaims

Let's get the PersistentVolumeClaims for web-0, web-1, and web-2:

```
$ kubectl get pvc -l app=nginx
NAME        STATUS   VOLUME                                     CAPACITY   ACCESS MODES   STORAGECLASS   AGE
www-web-0   Bound    pvc-1568146b-82ee-4658-969b-e4afbd660701   1Gi        RWO            standard       107m
www-web-1   Bound    pvc-d0505e50-ac97-4263-92ac-29d3ce97f2c1   1Gi        RWO            standard       107m
www-web-2   Bound    pvc-5a021388-5272-416b-825f-0883d2d4ff06   1Gi        RWO            standard       107m



```

The StatefulSet controller created three PersistentVolumeClaims that are bound to three PersistentVolumes. As the cluster used in this post is configured to dynamically provision PersistentVolumes, the PersistentVolumes were created and bound automatically.

The NGINX webservers, by default, will serve an index file at /usr/share/nginx/html/index.html. The volumeMounts field in the StatefulSets spec ensures that the /usr/share/nginx/htmll directory is backed by a PersistentVolume.

```




```










