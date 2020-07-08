---
layout: default
title: Networking with Calico
parent: Rancher Networking
nav_order: 11
---

# Networking with Calico


- Architecture

  - Calico operates at Layer 3 and assigns every workload a routable IP address. It prefers to operate by using BGP without an overlay network for the highest speed and efficiency, but in scenarios where hosts cannot directly communicate with one another, it can utilize an overlay solution such as VxLAN or 
IP in IP .
  - Calico supports network policies for protecting workloads and nodes from malicious activity or aberrant applications.
  - The Calico networking Pod contains a CNI container, a container that runs an agent that tracks Pod deployments and registers addresses and routes, and a daemon that announces the IP and route information to the network via the Border Gateway Protocol (BGP). The BGP daemons build a map of the network that enables cross-host communication.
  - Calico requires a distributed and fault-tolerant key/value datastore, and deployments often choose etcd to deliver this component. Calico uses it to store metadata about routes, virtual interfaces, and network policy objects. The Felix agent in the calico-node Pod communicates with etcd to publish this 
  information.  Calico can use a dedicated HA deployment of etcd, or it can use the Kubernetes etcd datastore via the Kubernetes API. Please see the Calico deployment documentation to understand the functional restrictions that are present when using the Kubernetes API for storing Calico data.
  - The final piece of a Calico deployment is the controller. Although presented as a single object, it is a set of controllers that run as a control loop within Kubernetes to manage policy, workload endpoints, and node changes
  
      -  The Policy Controller watches for changes in the defined network policies and translates them into Calico network policies.
      - The Profile Controller watches for the addition or removal of namespaces and programs Calico objects called Profiles.
      - Calico stores Pod information as workload endpoints. The Workload Endpoint Controller watches for updates to labels on the Pod and updates the workload endpoints.
      - The Node Controller loop watches for the addition or removal of Kubernetes nodes and updates the kvdb with the corresponding data.




- First lets setup the k8s cluster with kubeadm - this will take a minute - in this case we will be setting the pod network to a custom cidr since calico 
is already to use it, and downloading the images before executing the kubeadm init.
```

controlplane $ kubeadm config images pull
I0708 09:09:25.465243    7537 version.go:240] remote version is much newer: v1.18.5; falling back to: stable-1.14
[config/images] Pulled k8s.gcr.io/kube-apiserver:v1.14.10
[config/images] Pulled k8s.gcr.io/kube-controller-manager:v1.14.10
[config/images] Pulled k8s.gcr.io/kube-scheduler:v1.14.10
[config/images] Pulled k8s.gcr.io/kube-proxy:v1.14.10
[config/images] Pulled k8s.gcr.io/pause:3.1
[config/images] Pulled k8s.gcr.io/etcd:3.3.10
[config/images] Pulled k8s.gcr.io/coredns:1.3.1
controlplane $ kubeadm init --pod-network-cidr 192.168.0.0/16
I0708 09:09:40.981909    7684 version.go:240] remote version is much newer: v1.18.5; falling back to: stable-1.14
[init] Using Kubernetes version: v1.14.10
[preflight] Running pre-flight checks
[preflight] Pulling images required for setting up a Kubernetes cluster
[preflight] This might take a minute or two, depending on the speed of your internet connection
[preflight] You can also perform this action in beforehand using 'kubeadm config images pull'
[kubelet-start] Writing kubelet environment file with flags to file "/var/lib/kubelet/kubeadm-flags.env"
[kubelet-start] Writing kubelet configuration to file "/var/lib/kubelet/config.yaml"
[kubelet-start] Activating the kubelet service
[certs] Using certificateDir folder "/etc/kubernetes/pki"
[certs] Generating "ca" certificate and key
[certs] Generating "apiserver" certificate and key
[certs] apiserver serving cert is signed for DNS names [controlplane kubernetes kubernetes.default kubernetes.default.svc kubernetes.default.svc.cluster.local] and IPs [10.96.0.1 172.17.0.52]
[certs] Generating "apiserver-kubelet-client" certificate and key
[certs] Generating "front-proxy-ca" certificate and key
[certs] Generating "front-proxy-client" certificate and key
[certs] Generating "etcd/ca" certificate and key
[certs] Generating "etcd/healthcheck-client" certificate and key
[certs] Generating "apiserver-etcd-client" certificate and key
[certs] Generating "etcd/peer" certificate and key
[certs] etcd/peer serving cert is signed for DNS names [controlplane localhost] and IPs [172.17.0.52 127.0.0.1 ::1]
[certs] Generating "etcd/server" certificate and key
[certs] etcd/server serving cert is signed for DNS names [controlplane localhost] and IPs [172.17.0.52127.0.0.1 ::1]
[certs] Generating "sa" key and public key
[kubeconfig] Using kubeconfig folder "/etc/kubernetes"
[kubeconfig] Writing "admin.conf" kubeconfig file
[kubeconfig] Writing "kubelet.conf" kubeconfig file
[kubeconfig] Writing "controller-manager.conf" kubeconfig file
[kubeconfig] Writing "scheduler.conf" kubeconfig file
[control-plane] Using manifest folder "/etc/kubernetes/manifests"
[control-plane] Creating static Pod manifest for "kube-apiserver"
[control-plane] Creating static Pod manifest for "kube-controller-manager"
[control-plane] Creating static Pod manifest for "kube-scheduler"
[etcd] Creating static Pod manifest for local etcd in "/etc/kubernetes/manifests"
[wait-control-plane] Waiting for the kubelet to boot up the control plane as static Pods from directory "/etc/kubernetes/manifests". This can take up to 4m0s
[apiclient] All control plane components are healthy after 14.003282 seconds
[upload-config] storing the configuration used in ConfigMap "kubeadm-config" in the "kube-system" Namespace
[kubelet] Creating a ConfigMap "kubelet-config-1.14" in namespace kube-system with the configuration for the kubelets in the cluster
[upload-certs] Skipping phase. Please see --experimental-upload-certs
[mark-control-plane] Marking the node controlplane as control-plane by adding the label "node-role.kubernetes.io/master=''"
[mark-control-plane] Marking the node controlplane as control-plane by adding the taints [node-role.kubernetes.io/master:NoSchedule]
[bootstrap-token] Using token: m5hra3.v3aeq2gy24r3du5x
[bootstrap-token] Configuring bootstrap tokens, cluster-info ConfigMap, RBAC Roles
[bootstrap-token] configured RBAC rules to allow Node Bootstrap tokens to post CSRs in order for nodesto get long term certificate credentials
[bootstrap-token] configured RBAC rules to allow the csrapprover controller automatically approve CSRsfrom a Node Bootstrap Token
[bootstrap-token] configured RBAC rules to allow certificate rotation for all node client certificatesin the cluster
[bootstrap-token] creating the "cluster-info" ConfigMap in the "kube-public" namespace
[addons] Applied essential addon: CoreDNS
[addons] Applied essential addon: kube-proxy

Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 172.17.0.52:6443 --token m5hra3.v3aeq2gy24r3du5x \
    --discovery-token-ca-cert-hash sha256:814d6aa4b1161035436d5a5da86dcc070265c78660d94fc707baf70b44051f51

```

After kubeadmin completes we need to complete two other process' listed in the output of the above command:

    copy the config to the ~/.kube file, and
    run the join command on the second node.

(1). Copy (ctrl-insert) the 3 lines starting with mkdir and run in the top terminal (shift-insert), you'll then be able to run kubectrl. OR just run the following: 
```

$ mkdir -p $HOME/.kube ; sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config ;sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
(2). You'll see the join command with it's token at the end off the kubeadm init stdout, copy that and paste it into the second node in the bottom terminal. 
If you run kubectl get nodes in the top terminal you should see the two nodes running..
```
controlplane $ kubectl get nodes
NAME           STATUS     ROLES    AGE    VERSION
controlplane   NotReady   master   2m5s   v1.14.0
```
You'll see nodes are not totally ready since we don't have a network solution working.

And lets see what control plane pods are runnning: k get pods --all-namespaces Wait a minute or two, you should see all 8 pods up and running.
```
controlplane $ k get pods --all-namespaces
NAMESPACE     NAME                                   READY   STATUS    RESTARTS   AGE
kube-system   coredns-584795fc57-22dh2               0/1     Pending   0          3m49s
kube-system   coredns-584795fc57-zpm47               0/1     Pending   0          3m49s
kube-system   etcd-controlplane                      1/1     Running   0          3m1s
kube-system   kube-apiserver-controlplane            1/1     Running   0          2m49s
kube-system   kube-controller-manager-controlplane   1/1     Running   0          2m55s
kube-system   kube-proxy-6llvn                       1/1     Running   0          3m49s
kube-system   kube-scheduler-controlplane            1/1     Running   0          2m57s
```

Check Bridges

Let's take a look at the linux bridges on this system, first the master and then host 2

`brctl show`

and on the second host:

`brctl show`

and since we're using docker as the Container Runtime lets look at the configuration:

`docker network ls`

and on the second host:

`docker network ls`

Let's take a closer look at the docker bridge "bridge"

`docker network inspect bridge`

Note that under containers, there are no containers listed, and the IPAM settings Execute the same cmd on the lower terminal, you see the IPAM address range the same.

`docker network inspect bridge`

The first control plane componet to come up is the kubelet, this will be resposible for directly instructing the containers and the local network. and for the cluster network through a network solution like Calico. 
```
controlplane $ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.024210ccc817       no
controlplane $ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
8572998376ab        bridge              bridge              local
361ca442e405        host                host                local
e711d25d9678        none                null                local
controlplane $ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "8572998376abe8ae7aefe2e826d5672fdb603f3246b4fd63ca0baf46aec523fd",
        "Created": "2020-07-08T08:21:09.127727928Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.18.0.1/24",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]


```

```
node01 $ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.02425f3cbb10       no
node01 $ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
50aeca7d8bb5        bridge              bridge              local
361ca442e405        host                host                local
e711d25d9678        none                null                local
 node01 $ docker network inspect bridge
[
    {
        "Name": "bridge",
        "Id": "50aeca7d8bb5239e2353a2532b02705966ed2e423816a907d177dd8814aa8dbd",
        "Created": "2020-07-08T08:21:06.833044591Z",
        "Scope": "local",
        "Driver": "bridge",
        "EnableIPv6": false,
        "IPAM": {
            "Driver": "default",
            "Options": null,
            "Config": [
                {
                    "Subnet": "172.18.0.1/24",
                    "Gateway": "172.18.0.1"
                }
            ]
        },
        "Internal": false,
        "Attachable": false,
        "Ingress": false,
        "ConfigFrom": {
            "Network": ""
        },
        "ConfigOnly": false,
        "Containers": {},
        "Options": {
            "com.docker.network.bridge.default_bridge": "true",
            "com.docker.network.bridge.enable_icc": "true",
            "com.docker.network.bridge.enable_ip_masquerade": "true",
            "com.docker.network.bridge.host_binding_ipv4": "0.0.0.0",
            "com.docker.network.bridge.name": "docker0",
            "com.docker.network.driver.mtu": "1500"
        },
        "Labels": {}
    }
]
node01 $


```
Kubelet Configuration

Let's take a look at the startup kubelet config

`cat /var/lib/kubelet/config.yaml`

Or, look at the running config (which should be the same):

`ps -aux | grep /usr/bin/kubelet`

lets make it easier to read with the sed command:

`ps -aux | grep /usr/bin/kubelet | sed "s/--/\n--/g"`

note the lines

```
cat /var/lib/kubelet/config.yaml
address: 0.0.0.0
apiVersion: kubelet.config.k8s.io/v1beta1
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /etc/kubernetes/pki/ca.crt
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
cgroupDriver: cgroupfs
cgroupsPerQOS: true
clusterDNS:
- 10.96.0.10
clusterDomain: cluster.local
configMapAndSecretChangeDetectionStrategy: Watch
containerLogMaxFiles: 5
containerLogMaxSize: 10Mi
contentType: application/vnd.kubernetes.protobuf
cpuCFSQuota: true
cpuCFSQuotaPeriod: 100ms
cpuManagerPolicy: none
cpuManagerReconcilePeriod: 10s
enableControllerAttachDetach: true
enableDebuggingHandlers: true
enforceNodeAllocatable:
- pods
eventBurst: 10
eventRecordQPS: 5
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
evictionPressureTransitionPeriod: 5m0s
failSwapOn: true
fileCheckFrequency: 20s
hairpinMode: promiscuous-bridge
healthzBindAddress: 127.0.0.1
healthzPort: 10248
httpCheckFrequency: 20s
imageGCHighThresholdPercent: 85
imageGCLowThresholdPercent: 80
imageMinimumGCAge: 2m0s
iptablesDropBit: 15
iptablesMasqueradeBit: 14
kind: KubeletConfiguration
kubeAPIBurst: 10
kubeAPIQPS: 5
makeIPTablesUtilChains: true
maxOpenFiles: 1000000
maxPods: 110
nodeLeaseDurationSeconds: 40
nodeStatusReportFrequency: 1m0s
nodeStatusUpdateFrequency: 10s
oomScoreAdj: -999
podPidsLimit: -1
port: 10250
registryBurst: 10
registryPullQPS: 5
resolvConf: /etc/resolv.conf
rotateCertificates: true
runtimeRequestTimeout: 2m0s
serializeImagePulls: true
staticPodPath: /etc/kubernetes/manifests
streamingConnectionIdleTimeout: 4h0m0s
syncFrequency: 1m0s
volumeStatsAggPeriod: 1m0s
controlplane $ps -aux | grep /usr/bin/kubelet
root      7872  3.1  4.4 1056764 91212 ?       Ssl  09:09   0:18 /usr/bin/kubelet --bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf --kubeconfig=/etc/kubernetes/kubelet.conf --config=/var/lib/kubelet/config.yaml --cgroup-driver=systemd --network-plugin=cni --pod-infra-container-image=k8s.gcr.io/pause:3.1
controlplane $ ps -aux | grep /usr/bin/kubelet | sed "s/--/\n--/g"
root      7872  3.1  4.4 1056764 91212 ?       Ssl  09:09   0:19 /usr/bin/kubelet
--bootstrap-kubeconfig=/etc/kubernetes/bootstrap-kubelet.conf
--kubeconfig=/etc/kubernetes/kubelet.conf
--config=/var/lib/kubelet/config.yaml
--cgroup-driver=systemd
--network-plugin=cni
--pod-infra-container-image=k8s.gcr.io/pause:3.1
root     13069  0.0  0.0  14228   956 pts/1    S+   09:20   0:00 grep
--color=auto /usr/bin/kubelet
controlplane $



```
- --network plugin=cni
- --cni-bin-dir=/opt/cni/bin    # contains the availble plugins binaries for below
- --cni-conf-dir=/etc/cni/net.d # contains the conf files for network plugins to use (in the file name)

the kubelet looks for the right script to run on container creation with net.d, which points to the network script to run (netscript.sh with the container) (WIP: re-word this statement to be more correct)

Lets look at the cni config

```

controlplane $ ls /etc/cni/net.d
controlplane $ ls /opt/cni/bin
bridge  flannel      host-local  loopback  portmap  sample  vlan
dhcp    host-device  ipvlan      macvlan   ptp      tuning
controlplane $ k describe pod kube-apiserver-master  -n kube-system
controlplane $ k run nettools --image=busybox --replicas 2 -- sleep 3600
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nettools created
controlplane $

```

Setup Calico network

OK, lets download and install the calico manifest curl https://docs.projectcalico.org/v3.9/manifests/calico.yaml -O

`kubectl apply -f calico.yaml`

And check the pods and daemonsets are up:

`k get pods -n kube-system k get ds -n kube-system`

and the pods are controlled by rs' k get rs -n kube-system

Let's take a look at the logs for the calico pods k logs  -n kube-system -l k8s-app=calico-node

and the docker containers ps -aux | grep calico

```
controlplane $ curl https://docs.projectcalico.org/v3.9/manifests/calico.yaml -O
controlplane $ kubectl apply -f calico.yaml
configmap/calico-config created
customresourcedefinition.apiextensions.k8s.io/felixconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamblocks.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/blockaffinities.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamhandles.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ipamconfigs.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgppeers.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/bgpconfigurations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/ippools.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/hostendpoints.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/clusterinformations.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/globalnetworksets.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networkpolicies.crd.projectcalico.org created
customresourcedefinition.apiextensions.k8s.io/networksets.crd.projectcalico.org created
clusterrole.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrolebinding.rbac.authorization.k8s.io/calico-kube-controllers created
clusterrole.rbac.authorization.k8s.io/calico-node created
clusterrolebinding.rbac.authorization.k8s.io/calico-node created
daemonset.apps/calico-node created
serviceaccount/calico-node created
deployment.apps/calico-kube-controllers created
serviceaccount/calico-kube-controllers created
controlplane $ k get pods -n kube-system
NAME                                      READY   STATUS     RESTARTS   AGE
calico-kube-controllers-f67d5b96f-gvch9   0/1     Pending    0          3s
calico-node-vwmbn                         0/1     Init:0/3   0          3s
coredns-584795fc57-22dh2                  0/1     Pending    0          13m
coredns-584795fc57-zpm47                  0/1     Pending    0          13m
etcd-controlplane                         1/1     Running    0          12m
kube-apiserver-controlplane               1/1     Running    0          12m
kube-controller-manager-controlplane      1/1     Running    0          12m
kube-proxy-6llvn                          1/1     Running    0          13m
kube-scheduler-controlplane               1/1     Running    0          12m
controlplane $ k get ds -n kube-system
NAME          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR                 AGE
calico-node   1         1         0       1            0           beta.kubernetes.io/os=linux   5s
kube-proxy    1         1         1       1            1           <none>                        13m
controlplane $ k get rs -n kube-system
NAME                                DESIRED   CURRENT   READY   AGE
calico-kube-controllers-f67d5b96f   1         1         0       8s
coredns-584795fc57                  2         2         0       13m
controlplane $ k logs  -n kube-system -l k8s-app=calico-node
controlplane $ ps -aux | grep calico
root     17084  0.0  0.0  14228   972 pts/1    S+   09:23   0:00 grep --color=auto calico
controlplane $


```
