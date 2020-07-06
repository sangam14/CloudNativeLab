---
layout: default
title: Pod Networking 
parent: Rancher Networking
nav_order: 6
---

# Pod Networking 

The Pod is the smallest unit in Kubernetes, so it is essential to first understand Kubernetes networking in the context of communication between Pods.
Because a Pod can hold more than one container, we can start with a look at how communication happens between containers in a Pod.
Although Kubernetes can use Docker for the underlying container runtime, its approach to networking differs slightly and imposes some basic principles:
 -  Any Pod can communicate with any other Pod without the use of network address translation (NAT). To facilitate this, Kubernetes assigns each Pod an IP address that is routable within the cluster.
 -  A node can communicate with a Pod without the use of NAT.  
 -  A Pod's awareness of its address is the same as how other resources see the address. The host's address doesn't mask it.These principles give a unique and first-class identity to every Pod in the cluster.
Because of this, the networking model is more straightforward and does not need to include port mapping for the running container workloads.
By keeping the model simple, migrations into a Kubernetes cluster require fewer changes to the container and how it communicates.


# The Pause Container
- A piece of infrastructure that enables many networking features in Kubernetes is known as the pause container. 
- This container runs along side the containers defined in a Pod and is responsible for providing the network namespace that the other containers share. 
It is analogous to joining the network of another container that we described in the User Defined Network section above.
- The pause container was initially designed to act as the init process within a PID namespace shared by all containers in the Pod.
It performed the function of reaping zombie processes when a container died. PID namespace sharing is now disabled by default, so unless it has been explicitly enabled in the kubelet, all containers run their process as PID 1.


If we launch a Pod running Nginx, we can inspect the Docker container running within the Pod.

```
 kubectl run nginx --image=nginx 
kubectl run --generator=deployment/apps.v1 is DEPRECATED and will be removed in a future version. Use kubectl run --generator=run-pod/v1 or kubectl create instead.
deployment.apps/nginx created
sangam:~ sangam$ kubectl get pods -o wide | grep nginx
nginx-7bb7cd8db5-fxf9w               1/1     Running                      0          60s     10.4.63.141   gke-us-central1-cloud-okteto-com-dev-520923cd-17sm   <none>           <none>

```
When we do so, we see that the container does not have the network settings provided to it. The pause container which runs as part of the Pod is the one which gives the networking constructs to the Pod

Note: Run the commands below on the host where the nginx Pod is scheduled

```

node01 $ docker ps | grep nginx
4db98393f5ff        nginx                                          "/docker-entrypoint.…"   50 secondsago       Up 38 seconds                           k8s_nginx_nginx_default_7422cd8a-7f56-4b9d-abee-3158e84feb87_0
0c51920d3349        k8s.gcr.io/pause:3.2                           "/pause"                 59 secondsago       Up 58 seconds                           k8s_POD_nginx_default_7422cd8a-7f56-4b9d-abee-3158e84feb87_7


```
```
node01 $ docker inspect 4db98393f5ff  --format="{{json .NetworkSettings}}"
{"Bridge":"","SandboxID":"","HairpinMode":false,"LinkLocalIPv6Address":"","LinkLocalIPv6PrefixLen":0,"Ports":{},"SandboxKey":"","SecondaryIPAddresses":null,"SecondaryIPv6Addresses":null,"EndpointID":"","Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"IPAddress":"","IPPrefixLen":0,"IPv6Gateway":"","MacAddress":"","Networks":{}}

```
```
node01 $ docker inspect 0c51920d3349   --format="{{json .NetworkSettings}}"
{"Bridge":"","SandboxID":"d0808905b62dd655b22a9927e7a5f5f2ad00ab5331765596bb24fa5ff93240ed","HairpinMode":false,"LinkLocalIPv6Address":"","LinkLocalIPv6PrefixLen":0,"Ports":{},"SandboxKey":"/var/run/docker/netns/d0808905b62d","SecondaryIPAddresses":null,"SecondaryIPv6Addresses":null,"EndpointID":"","Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"IPAddress":"","IPPrefixLen":0,"IPv6Gateway":"","MacAddress":"","Networks":{"none":{"IPAMConfig":null,"Links":null,"Aliases":null,"NetworkID":"419c5963169c9ffa00c35371ac71de4a6eee8985b0bb7d8c71a0aac403c5d40b","EndpointID":"74757167a48d43352773406ec7c83ba1a824cb9b2f4cc0875e1b7aaa3c73dc38","Gateway":"","IPAddress":"","IPPrefixLen":0,"IPv6Gateway":"","GlobalIPv6Address":"","GlobalIPv6PrefixLen":0,"MacAddress":"","DriverOpts":null}}}

```
# Intra-Pod Communication
- Kubernetes follows the IP-per-Pod model where it assigns a routable IP address to the Pod. The containers within the Pod share the same network space and communicate with one another over localhost. Like processes running on a host, two containers cannot each use the same network port, but we can work around this by changing the manifest.

# Inter-Pod Communication
- Because it assigns routable IP addresses to each Pod, and because it requires that all resources see the address of a Pod the same way, Kubernetes assumes that all Pods communicate with one another via their assigned addresses. Doing so removes the need for an external service discovery mechanism

