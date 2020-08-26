---
layout: default
title: Docker Swarm
parent: Docker Fundamental
nav_order: 5
---

# What is Docker swarm?

- You probably have not noticed this, but so far, all of the Docker workstation deployments, or nodes that we have used in our examples have 
been run in single-engine mode. What does that mean? Well, it tells us that the Docker installation is managed directly and as a 
standalone Docker environment. While this is effective, it is not very efficient and it does not scale well. Of course, 
Docker understands the limitations and has provided a powerful solution to this problem. It is called Docker swarm. 
- Docker swarm is a way to link Docker nodes together, and manage those nodes and the dockerized applications that run on them efficiently and at scale. 
Simply stated, a Docker swarm is a group of Docker nodes connected and managed as a cluster or swarm. Docker swarm is built into the Docker engine, 
so no additional installation is required to use it. When a Docker node is part of a swarm, it is running in swarm mode. If there is any doubt, 
you can easily check whether a system running Docker is part of a swarm or is running in single-engine mode using the `docker system info` command: <br> 

single engine mode 
```
node1$ docker system info | grep Swarm 
Swarm: inactive 
```
swarm mode 

```
node1$ docker system info | grep Swarm 
Swarm: active 

```

- The features that provide swarm mode are part of the Docker SwarmKit, which is a tool for orchestrating distributed systems at scale, that is, 
Docker swarm clusters. Once a Docker node joins a swarm, it becomes a swarm node, becoming either a Manager node or a Worker node. 
We will talk about the difference between managers and workers shortly. For now, know that the very first Docker node to join a new swarm becomes the first Manager, also known as the Leader. There is a lot of technical magic that happens when that first node joins a swarm (actually, it creates and initializes the swarm, and then joins it) 
and becomes the leader. Here is some of the wizardry that happens (in no particular order):

   - A Swarm-ETCD-based configuration database or cluster store is created and encrypted <br> 
   - Mutual TLS (mTLS) authentication and encryption is set up for all inter-node communication <br> 
   - Container orchestration is enabled, which takes responsibility for managing which containers run on which nodes <br> 
   - The cluster store is configured to automatically replicate to all manager nodes <br> 
   - The node gets assigned a cryptographic ID <br> 
   - A Raft-based distributed consensus-management system is enabled <br> 
   -  The node becomes a Manager and is elected to the status of swarm leader <br> 
   -  The swarm managers are configured for HA <br> 
   -  A public-key infrastructure system is created <br> 
   -  The node becomes the certificate authority, allowing it to issue client certificates to any nodes that join the swarm <br> 
   -  A default 90-day certificate-rotation policy is configured on the certificate authority <br> 
   
## The node gets issued its client certificate, which includes its name, ID, the swarm ID, and the node's role in the swarm
    
- Creating a new cryptographic join token for adding new swarm managers occurs
- Creating a new cryptographic join token for adding new swarm workers occurs

That list represents a lot of powerful features that you get by joining the first node to a swarm. And, 
with great power comes great responsibility, meaning that you really need to be prepared to do a lot of work to create your Docker swarm,
as you might well imagine. So, let's move on to the next section, where we will discuss how to enable all of these features when you set up a swarm cluster.

- more info :
   - The repository for SwarmKit: (https://github.com/docker/swarmkit)
   - The Raft consensus algorithm: (https://raft.github.io/)
    
    
# How to set up a Docker swarm cluster

incredible features that get enabled and set up when you create a Docker swarm cluster. So, 
now I am going to show you all of the steps needed to set up a Docker swarm cluster. Are you ready? Here they are:

```
# Set up your Docker swarm cluster
docker swarm init

```
- What? Wait? Where is the rest of it? Nope. There is nothing missing. 
All of the setup and functionality that is described in the preceding section is achieved with one simple command.
With that single swarm init command, the swarm cluster is created, the node is transformed from a single-instance node into a swarm-mode node, 
the role of manager is assigned to the node and it is elected as the leader of the swarm, the cluster store is created, the node
becomes the certificate authority of the cluster and assigns itself a new certificate that includes a cryptographic ID, a 
new cryptographic join token is created for managers, and another is created for workers, and on and on. This is complexity made simple.

- The swarm commands make up another Docker management group. Here are the swarm-management commands:

```
$ docker swarm  --help

Usage:  docker swarm COMMAND

Manage Swarm

Commands:
  ca          Display and rotate the root CA
  init        Initialize a swarm
  join        Join a swarm as a node and/or manager
  join-token  Manage join tokens
  leave       Leave the swarm
  unlock      Unlock swarm
  unlock-key  Manage the unlock key
  update      Update the swarm

Run 'docker swarm COMMAND --help' for more information on a command.


```

- your Docker nodes to allow Docker swarm to function properly. Here is the information straight from Docker's Getting started with swarm mode wiki:

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/docker-swarm.png)

- Two other ports that you may need to open for the REST API are as follows:
    -  TCP 2375 for Docker REST API (plain text)
    -  TCP 2376 for Docker REST API (ssl)
    
##  docker swarm init

Create the swarm cluster, add (this) the first Docker node to it, and then set up and enable all of the swarm features we just covered. 
The init command can be as simple as using it with no parameters, but there are many optional parameters available to fine-tune the initialization process. 
You can get a full list of the optional parameters, as usual, by using `--help`, but let's consider a few of the available parameters now:

- `--autolock`: Use this parameter to enable manager autolocking.
-  `--cert-expiry` duration: Use this parameter to change the default validity period (of 90 days) for node certificates.
- `--external-ca external-ca`: Use this parameter to specify one or more certificate-signing endpoints, that is, external CAs.


## docker swarm join-token

When you initialize the swarm by running the swarm init command on the first node, one of the functions that is executed creates unique
cryptographic join tokens, one joins additional manager nodes, and one joins worker nodes. Using the `join-token` command, you can obtain these two join tokens.
In fact, using the `join-token` command will deliver the full join command for whichever role you specify. The role parameter is required. 
Here are examples of the command:

```
# Get the join token for adding managers
docker swarm join-token manager
# Get the join token for adding workers
docker swarm join-token worker

```


```
$ docker swarm join-token manager
To add a manager to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2n9grlv2pjg0wh5k3nigli8ho8mxse5gh9a1vygjogcg3kq9n9-ahwwzbj6g234yturiu6ko7nwq 192.168.0.28:2377
    
$ docker swarm join-token worker
To add a worker to this swarm, run the following command:

    docker swarm join --token SWMTKN-1-2n9grlv2pjg0wh5k3nigli8ho8mxse5gh9a1vygjogcg3kq9n9-3i7o1d5qk4cq8lxl6r0wa4ctb 192.168.0.28:2377    

```
  
  
```  
# Rotate the worker join token
docker swarm join-token --rotate worker

```
- Note that this does not invalidate existing workers that have used the old, now invalid, join token. They are still a part of the swarm and are unaffected by the change in the join token.
  Only new nodes that you wish to join to the swarm need to use the new token.


## docker swarm join
- You have already seen the join command used in the preceding docker swarm join-token section. The join command is used, in conjunction with a cryptographic join token, to add a Docker node to the swarm. All nodes except the very first node will use the join command to become part of the swarm (the first node uses the "init" command, of course). The join command has a few parameters, the most important of them being the --token parameter.
This is the required join token, obtainable with the `join-token` command. Here is an example:

```
# Join this node to an existing swarm
docker swarm join --token SWMTKN-1-3ovu7fbnqfqlw66csvvfw5xgljl26mdv0dudcdssjdcltk2sen-a830tv7e8bajxu1k5dc0045zn 192.168.159.156:2377

```
- You will notice that the role is not needed for this command. This is because the token itself is associated with the role it has been created for.
When you execute the join, the output provides an informational message telling you what role the node has joined as manager or worker.
If you have inadvertently use a manager token to join a worker or vice versa, you can use the `leave` command to remove a node from the swarm, and then using the token for the actual desired role,
rejoin the node to the swarm.


## docker swarm ca


- The swarm ca command is used when you want to view the current certificate for the swarm, or you need to rotate the current swarm certificate. 
To rotate the certificate, you would include the `--rotate` parameter:

```
# View the current swarm certificate
docker swarm ca
# Rotate the swarm certificate
docker swarm ca --rotate

```

The `swarm ca` command can only be executed successfully on a swarm manager node. One reason you might use the rotate swarm certificate
feature is if you are moving from the internal root CA to an external CA, or vice versa. Another reason you might need to rotate the swarm certificate is 
in the event of one or more manager nodes getting compromised. In that case, rotating the swarm certificate will block all other managers from being able
to communicate with the manager that rotated the certificate or each other using the old certificate. When you rotate the certificate, the command will 
remain active, blocking until all swarm nodes, both managers and workers, have been updated. Here is an example of rotating the certificate on a very small cluster:


```
$ docker swarm ca --rotate
desired root digest: sha256:b08295a52e50de8c774134dd9c7eb5d65070496a96800bfa9908f
  rotated TLS certificates:  1/1 nodes
  rotated CA certificates:   1/1 nodes
-----BEGIN CERTIFICATE-----
MIIBazCCARCgAwIBAgIURTYtW0nyF02ryQevIRDEQFfJs60wCgYIKoZIzj0EAwIw
EzERMA8GA1UEAxMIc3dhcm0tY2EwHhcNMjAwODI2MDQwMjAwWhcNNDAwODIxMDQw
MjAwWjATMREwDwYDVQQDEwhzd2FybS1jYTBZMBMGByqGSM49AgEGCCqGSM49AwEH
A0IABHgMnncWfRr/+kRcwJmNoRVRIxIHKRZfkiuWGSU3PAzwfvi9np67fA2K6ers
etS1wWMdAW10j7pzsHeaBvJ3/b+jQjBAMA4GA1UdDwEB/wQEAwIBBjAPBgNVHRMB
Af8EBTADAQH/MB0GA1UdDgQWBBRxBM0kEVLrwGRxARPef7uMpBVs2TAKBggqhkjO
PQQDAgNJADBGAiEA0H6ydFFi+chjpY3xrdHT2yEucHVwxJz+pG5MCu8triUCIQC6
puXsir+4Py3nA0jYR0/G9gAMoZFABYYRqpuo790HdQ==
-----END CERTIFICATE-----
[manager1] (local) root@192.168.0.12 ~

```

Since the command will remain active until all nodes have updated both the TLS certificate and the CA certificate, it can present an issue if there are nodes in the swarm that are offline. When that is a potential problem, you can include the --detach parameter, and the command will initiate the certificate rotation and return control immediately to the session. Be aware that you will not get any status as to the progress, success, or failure of the certificate rotation when you use the --detach optional parameter.
You can use the node ls command to query the state of the certificates within the cluster to check the progress. Here is the full command you can use:

```
# Query the state of the certificate rotation in a swarm cluster
docker node ls --format '{{.ID}} {{.Hostname}} {{.Status}} {{.TLSStatus}}'
```

The ca rotate command will continue trying to complete, either in the foreground, or in the background if detached. 
If a node was offline when the rotate is initiated, and it comes back online, the certificate rotation will complete.
Here is an example of node01 being offline when the rotate command was executed, and then a while later,  after it came back on; check the status found it successfully rotated:

```
$ docker swarm ca --rotate --detach
[manager1] (local) root@192.168.0.12 ~
$ docker node ls --format '{{.ID}} {{.Hostname}} {{.Status}} {{.TLSStatus}}'
o6i68c180mr29tajk4sf19qyk manager1 Ready Ready

```

Another important point to remember is that rotating the certificate will immediately invalidate both of the current join tokens.


## docker swarm unlock





