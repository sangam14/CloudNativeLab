
# Bridge Networking


- In a standard Docker installation, the Docker daemon creates a bridge on the host with the name of docker0. 
When a container launches, Docker then creates a virtual ethernet device for it. This device appears within the container as `eth0` and on 
the host with a name like vethxxxwhere xxx is a unique identifier for the interface. The `vethxxx `interface is added to the docker0 bridge,
and this enables communication with other containers on the same host that also use the default bridge. To demonstrate using the default bridge, 
run the following command on a host with Docker installed.
Since we are not specifying the network - the container will connect to the default bridge when it launches.

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/R-Bridge-Networking.png)

Run the `ip addr` and `ip route `commands inside of the container.  You will see the IP address of the container with the `eth0` interface:

```
$ docker run -it --rm busybox /bin/sh
/ # ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
7: eth0@if8: <BROADCAST,MULTICAST,UP,LOWER_UP,M-DOWN> mtu 1500 qdisc noqueue 
    link/ether 02:42:ac:11:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.17.0.2/16 brd 172.17.255.255 scope global eth0
       valid_lft forever preferred_lft forever
/ # 



```
n another terminal connected to the host, run the ip addr command. You will see the corresponding interface created for the container.
In the image below it is named veth5dd2b68@if9. Yours will be different.

```
$ ip addr | grep -A 50 veth

```
In another terminal connected to the host, run the ip addr command. You will see the corresponding interface created for the container. In the image below it is named veth5dd2b68@if9. Yours will be different.Although Docker mapped the container IPs on the bridge, network services running inside of the container are not visible outside of the host. To make them visible, the Docker Engine must be told when launching a container to map ports from that container to ports on the host. This process is called publishing. 
For example, if you want to map port 80 of a container to port 8080 on the host, then you would have to publish the port as shown in the following command:

```
docker run --name nginx -p 8080:80 nginx
Unable to find image 'nginx:latest' locally
latest: Pulling from library/nginx
8559a31e96f4: Pull complete 
8d69e59170f7: Pull complete 
3f9f1ec1d262: Pull complete 
d1f5ff4f210d: Pull complete 
1e22bfa8652e: Pull complete 
Digest: sha256:21f32f6c08406306d822a0e6e8b7dc81f53f336570e852e25fbe1e3e3d0d0133
Status: Downloaded newer image for nginx:latest
/docker-entrypoint.sh: /docker-entrypoint.d/ is not empty, will attempt to perform configuration
/docker-entrypoint.sh: Looking for shell scripts in /docker-entrypoint.d/
/docker-entrypoint.sh: Launching /docker-entrypoint.d/10-listen-on-ipv6-by-default.sh
10-listen-on-ipv6-by-default.sh: Getting the checksum of /etc/nginx/conf.d/default.conf
10-listen-on-ipv6-by-default.sh: Enabled listen on IPv6 in /etc/nginx/conf.d/default.conf
/docker-entrypoint.sh: Launching /docker-entrypoint.d/20-envsubst-on-templates.sh
/docker-entrypoint.sh: Configuration complete; ready for start up


```
In another terminal connected to the host, run the ip addr command. You will see the corresponding interface created for the container. In the image below it is named veth5dd2b68@if9. Yours will be different.Although Docker mapped the container IPs on the bridge, network services running inside of the container are not visible outside of the host. To make them visible, the Docker Engine must be told when launching a container to map ports from that container to ports on the host. This process is called publishing. For example, if you want to map port 80 of a container to port 8080 on the host, then you would have to publish the port as shown in the following command:docker run --name nginx -p 8080:80 nginxBy default, the Docker container can send traffic to any destination. The Docker daemon creates a rule within Netfilter that modifies outbound packets and changes the source address to be the address of the host itself. The Netfilter configuration allows inbound traffic via the rules that Docker creates when initially publishing the container's ports.The output included below 
shows the Netfilter rules created by Docker when it publishes a container’s ports.

```
$ docker run -p 8080:80 --name web -d nginx
39c2b39997cfd41897b21f0990ca63e4da3e5ff657c2399b82810d15fda5f208

$ docker inspect web --format='{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}'
172.17.0.2

```

```
sudo iptables -L -t nat -v
Chain PREROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    1    60 DOCKER     all  --  any    any     anywhere             anywhere             ADDRTYPE match dst-type LOCAL

Chain INPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain OUTPUT (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
   20  1380 DOCKER_OUTPUT  all  --  any    any     anywhere             127.0.0.11          
    0     0 DOCKER     all  --  any    any     anywhere            !127.0.0.0/8          ADDRTYPE match dst-type LOCAL

Chain POSTROUTING (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 MASQUERADE  all  --  any    !docker0  172.17.0.0/16        anywhere            

   20  1380 DOCKER_POSTROUTING  all  --  any    any     anywhere             127.0.0.11          
    0     0 MASQUERADE  tcp  --  any    any     172.17.0.2           172.17.0.2           tcp dpt:http

Chain DOCKER (2 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  docker0 any     anywhere             anywhere            
    0     0 DNAT       tcp  --  !docker0 any     anywhere             anywhere             tcp dpt:http-alt to:172.17.0.2:80

Chain DOCKER_OUTPUT (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DNAT       tcp  --  any    any     anywhere             127.0.0.11           tcp dpt:domain to:127.0.0.11:45406
   20  1380 DNAT       udp  --  any    any     anywhere             127.0.0.11           udp dpt:domain to:127.0.0.11:43934

Chain DOCKER_POSTROUTING (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 SNAT       tcp  --  any    any     127.0.0.11           anywhere             tcp spt:45406 to::53
```

```


sudo iptables -L -t  filter -v
Chain INPUT (policy ACCEPT 2077 packets, 370K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain FORWARD (policy ACCEPT 0 packets, 0 bytes)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-USER  all  --  any    any     anywhere             anywhere            
    0     0 DOCKER-ISOLATION-STAGE-1  all  --  any    any     anywhere             anywhere            
    0     0 ACCEPT     all  --  any    docker0  anywhere             anywhere             ctstate RELATED,ESTABLISHED
    0     0 DOCKER     all  --  any    docker0  anywhere             anywhere            
    0     0 ACCEPT     all  --  docker0 !docker0  anywhere             anywhere            
    0     0 ACCEPT     all  --  docker0 docker0  anywhere             anywhere            

Chain OUTPUT (policy ACCEPT 1592 packets, 2052K bytes)
 pkts bytes target     prot opt in     out     source               destination         

Chain DOCKER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 ACCEPT     tcp  --  !docker0 docker0  anywhere             172.17.0.2           tcp dpt:http

Chain DOCKER-ISOLATION-STAGE-1 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DOCKER-ISOLATION-STAGE-2  all  --  docker0 !docker0  anywhere             anywhere            
    0     0 RETURN     all  --  any    any     anywhere             anywhere            

Chain DOCKER-ISOLATION-STAGE-2 (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 DROP       all  --  any    docker0  anywhere             anywhere            
    0     0 RETURN     all  --  any    any     anywhere             anywhere            

Chain DOCKER-USER (1 references)
 pkts bytes target     prot opt in     out     source               destination         
    0     0 RETURN     all  --  any    any     anywhere             anywhere     





```

NAT table within Netfilter

