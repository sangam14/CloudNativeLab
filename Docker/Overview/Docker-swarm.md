---
layout: default
title: Docker Swarm
parent: Docker Fundamental
nav_order: 6
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

- You may recall from the discussion regarding the docker swarm init command that one of the optional parameters that you can include with the init command is --autolock. Using this parameter will enable the autolock feature on the swarm cluster. What does that mean? Well, when a swarm cluster is configured to use auto-locking, any time the docker daemon of a manager node goes offline, and then comes back online (that is, is restarted) it is necessary to enter an unlock key to allow the node to rejoin the swarm. Why would you use the auto-lock feature to lock your swarm? The auto-lock feature helps to protect the mutual TLS encryption key of the swarm, along with the encrypt and decrypt keys used with the swarm's raft logs. It is an additional security feature intended to supplement Docker Secrets. When the docker daemon restarts on the manager node of a locked swarm, you must enter the unlock key. Here is what using the unlock key looks like:


```
$ docker service docker restart 

$ docker swarm unlock 
please enter unlock key
$docker node ls
ID                            HOSTNAME            STATUS              AVAILABILITY        MANAGER STATUS      ENGINE VERSION
llt33mev2jndnye35bt3jpd7j *   manager1            Ready               Active              Reachable           19.03.11
msh0nhyi1whl4o1th4oua7l3s     manager2            Ready               Active              Reachable           19.03.11
qvgyd9lxbd90wj6rdm1cui41h     manager3            Ready               Active              Leader              19.03.11
vnqiv2t6z0awe38wdy6id2v0l     worker1             Ready               Active                                  19.03.11
fm3p3sa38z9brn3yr9ehynxak     worker2             Ready               Active                                  19.03.11
[manager1] (local) root@192.168.0.12 ~
$ 

```
- By the way, to the rest of the swarm, a manager node that has not been unlocked will report as down, even though the docker daemon is running. The swarm auto-lock feature can be enabled or disabled on an existing swarm cluster using the swarm update command, which we will take a look at shortly. The unlock key is generated during the swarm initialization and will be presented on the command line at that time. If you have lost the unlock key, you can retrieve it on an unlocked manager node using the swarm unlock-key command.

## docker swarm unlock-key

The `swarm unlock-key` command is much like the swarm ca command. The unlock-key command can be used to retrieve the current swarm unlock key, or it can be used to rotate the unlock key to a new one:

```
# Retrieve the current unlock key
docker swarm unlock-key
# Rotate to a new unlock key
docker swarm unlock-key --rotate

```

Depending on the size of the swarm cluster, the unlock key rotation can take a while for all of the manager nodes to get updated.

Note :- It is a good idea to keep the current (old) key handy for a while when you rotate the unlock key, on the off-chance that a manager node goes offline before getting the updated key. That way, you can still unlock the node using the old key. Once the node is unlocked and receives the rotated (new) unlock key, the old key can be discarded.

- As you might expect, the swarm unlock-key command is only useful when issued on a manager node of a cluster with the auto-lock feature enabled. If you have a cluster that does not have the auto-lock feature enabled, you can enable it with the swarm update command.

## docker swarm update

- There are several swarm cluster features that are enabled or configured when you initialize the cluster on the first manager node via the docker swarm init command. There may be times that you want to change which features are enabled, disabled, or configured after the cluster has been initialized. To accomplish this, you will need to use the swarm update command. For example, you may want to enable the auto-lock feature for your swarm cluster. Or, you might want to change the length of time that certificates are valid for. These are the types of changes you can execute using the `swarm update` command. Doing so might look like this:

```
# Enable autolock on your swarm cluster
docker swarm update --autolock=true
# Adjust certificate expiry to 30 days
docker swarm update --cert-expiry 720h

```

Here is the list of settings that can be affected by the swarm update command:

```
$ docker swarm update --help

Usage:  docker swarm update [OPTIONS]

Update the swarm

Options:
      --autolock                        Change manager autolocking setting
                                        (true|false)
      --cert-expiry duration            Validity period for node
                                        certificates (ns|us|ms|s|m|h)
                                        (default 2160h0m0s)
      --dispatcher-heartbeat duration   Dispatcher heartbeat period
                                        (ns|us|ms|s|m|h) (default 5s)
      --external-ca external-ca         Specifications of one or more
                                        certificate signing endpoints
      --max-snapshots uint              Number of additional Raft
                                        snapshots to retain
      --snapshot-interval uint          Number of log entries between Raft
                                        snapshots (default 10000)
      --task-history-limit int          Task history retention limit
                                        (default 5)

```

## docker swarm leave

- This one is pretty much what you would expect. You can remove a docker node from a swarm with the leave command. Here is an example of needing to use the leave command to correct a user error:

```
$ docker system info | grep Swarm 
inactive 
$ docker swarm join --token SWMTKN-1-3ovu7fbnqfqlw66csvvfw5xgljl26mdv0dudcdssjdcltk2sen-a830tv7e8bajxu1k5dc0045zn 192.168.159.156:2377
docker system info | grep Swarm 
active 
$ docker swarm leave 
$ docker system info | grep Swarm 
inactive
```   

- for more details :- 
   - [Getting started with swarm mode tutorial](https://docs.docker.com/engine/swarm/swarm-tutorial/)
   - [The docker swarm init command wiki doc](https://docs.docker.com/engine/reference/commandline/swarm_init/)
   - [The docker swarm ca command wiki doc](https://docs.docker.com/engine/reference/commandline/swarm_ca/)
   - [The docker swarm join-token command wiki doc](https://docs.docker.com/engine/reference/commandline/swarm_join-token/)
   - [The docker swarm join command wiki doc](https://docs.docker.com/engine/reference/commandline/swarm_join/)
   - [The docker swarm unlock command wiki doc](https://docs.docker.com/engine/reference/commandline/swarm_unlock/)
   - [The docker swarm unlock-key command wiki doc](https://docs.docker.com/engine/reference/commandline/swarm_unlock-key/)
   - [The docker swarm update command wiki doc](https://docs.docker.com/engine/reference/commandline/swarm_update/)
   - [The docker swarm leave command wiki doc](https://docs.docker.com/engine/reference/commandline/swarm_leave/)
   - [Learn more about Docker Secrets](https://docs.docker.com/engine/swarm/secrets/)

# Managers and workers

- All manager nodes are actually worker nodes as well by default. This means that they can and will run containers. If you want to keep your managers from running workloads, you need to change the node's availability setting. Changing it to draining will carefully stop any running containers on the manager node marked as draining, and will start up those containers on other (non-draining) nodes. No new container workloads will be started on a node in drain mode, for example as follows:

```
# Set node03's availability to drain
docker node update --availability drain ubuntu-node03

```

There may be times when you want or need to change the role of a docker node in the swarm. You can promote a worker node to manager status, or you can demote a manager node to worker status. Here are some examples of these activities:

```
# Promote worker nodes 04 and 05 to manager status
docker node promote ubuntu-node04 ubuntu-node05
# Demote manager nodes 01 and 02 to worker status
docker node demote ubuntu-node01 ubuntu-node02

```
[Check out the official documentation on how nodes work](https://docs.docker.com/engine/swarm/how-swarm-mode-works/nodes/)


# Swarm services

- Docker swarm cluster, and how its nodes go from single-engine mode into swarm mode. You also know that the significance of that is to free you from directly managing individual running containers. So, you may be starting to wonder, if I don't manage my containers directly and individually now, how do I manage them? You've come to the right place! This is where swarm services come into play. swarm services allow you to define the desired state for your container application in terms of how many concurrent running copies of the container there should be. Let's take a quick look at what commands are available to us in the management group for swarm services, and then we'll talk about those commands:

```
docker service --help

Usage:  docker service COMMAND

Manage services

Commands:
  create      Create a new service
  inspect     Display detailed information on one or more services
  logs        Fetch the logs of a service or task
  ls          List services
  ps          List the tasks of one or more services
  rm          Remove one or more services
  rollback    Revert changes to a service's configuration
  scale       Scale one or multiple replicated services
  update      Update a service

Run 'docker service COMMAND --help' for more information on a command.

```

- The first thing that you'll probably want to do is create a new service, so we will begin our swarm services discussion with the service create command. 
  Here is the syntax and a basic sample of the service create command:

```
# Syntax for the service create command
# Usage: docker service create [OPTIONS] IMAGE [COMMAND] [ARG...]
# Create a service
docker service create --replicas 1 --name submarine alpine ping google.com

```

Let's break down the sample `service create` command shown here. First, you have the management group service followed by the create command. Then, we start getting into the parameters; the first one is `--replicas`. This defines the number of copies of the container that should be run concurrently. Next, we have the `--name` parameter. This one is pretty obvious and is the name of the service we are creating, in this case, submarine. We will be able to use the stated name in other service commands. After the name parameter, we have the fully-qualified Docker image name. In this case, it is just alpine. It could have been something such as `alpine:3.8`, or alpine:latest, or something more qualified such as `tenstartups/alpine:latest`. Following the image name to use for the service is the command to use when running the container and the parameters to pass to that command— `ping` and `google.com`, respectively. So, the preceding sample service create command will launch a single container from the alpine image, which will run the ping command with the google.com parameter, and then name the service submarine. Here is what that looks like:

```
# Create a service

$ docker service create --replicas 1 --name submarine alpine ping google.com
hb3wsapz67910mc2x44dricq1
overall progress: 0 out of 1 tasks 
overall progress: 1 out of 1 tasks 
1/1: running   
verify: Service converged 
```

You now know the basics of creating `docker services.` But, before you get too excited, there's still a lot of ground to cover for the service create command. In fact, this command has so many options that listing them all out would take two pages in this book. So, rather than do that, I want you to use the `--help` feature and enter the following command now:

```
# Get help with the service create command
docker service create --help

```

- Just so you know, the two parameters we used so far, `--replicas` and `--name`, are both optional. If you don't provide a number of replicas to use, the service will be created with a default of 1. Also, if you don't provide a name for the service, a fanciful name will be made up and given to the service. 
This is the same type of default naming we saw when using the docker container run command, It is generally better to provide both of these options for each
service create command issued.

- Also, know that generally speaking, the command and command parameters for the image that were supplied in the preceding sample are optional as well. In this specific case, they are necessary because, by itself, a container run from the alpine image with no other command or parameters supplied will just exit. In the sample, that would show up as a failure to converge the service and Docker would perpetually try to restart the service. Stated another way, you can leave off the command and its parameters if the image being used has them built in (such as in the `CMD` or `ENTRYPOINT` instruction of the Dockerfile).

- Let's move on to some more create parameters now. You should recall Learning Docker Commands that there is a `--publish `parameter you can use on a docker container run command that defines the port exposed on the docker host and the port in the container that the host port is mapped to. It looked something like this:

```
# Create a nginx web-server that redirects host traffic from port 8080 to port 80 in the container
docker container run --detach --name web-server1 --publish 8080:80 nginx
```

Well, you need the same functionality for a swarm service, and in their wisdom, Docker made the parameter used for both the container run command and the service create command the same: --publish. You can use the same abbreviated format we saw before, `--publish 8080:80`, or you can use a more verbose format: `--publish published=8080`, `target=80`. This still translates to redirect host traffic from port 8080 to port 80 in the container. Let's try out another example, this time one that uses the `--publish` parameter. We'll give the nginx image another run:

```
# Create a nginx web-server service using the publish parameter
docker service create --name web-service --replicas 3 --publish published=8080,target=80 nginx

```
This example will create a new service that runs three container replicas, using the nginx image and exposing port 80 on the containers and port 8080 on the hosts. Have a look:

```
$ docker service create --name web-service --replicas 3 --publish published=8080,target=80 nginx
bk5xisgb6vby5lii7mu9evbt7
overall progress: 3 out of 3 tasks 
1/3: running   
2/3: running   
3/3: running   
verify: Service converged 

```
Let's review some of the other service commands you will want to know and use. Once you've created some services, you might want a list of those services. This can be achieved with the `service list` command. It looks like this:

```
# List services in the swarm
# Usage: docker service ls [OPTIONS]
docker service list

```

Once you have reviewed the list of running services, you might want more details about one or more of those services. To achieve this, you would use the `service ps ` command. Have a look:

```
# List the tasks associated with a service
# Usage: docker service ps [OPTIONS] SERVICE [SERVICE...]
docker service ps

```
Once a service has outlived its usefulness, you might want to terminate it. The command to do that is the service remove command. Here is what that looks like:

```
# Remove one or more services from the swarm
# Usage: docker service rm SERVICE [SERVICE...]
docker service remove sleepy_snyder

```

If you want to remove all of the services running in the swarm, you can combine some of these commands and execute something such as this:

```
# Remove ALL the services from the swarm
docker service remove $(docker service list -q)

```

Finally, if you realize that the number of replicas currently configured is not set to the desired number, you can use the service scale command to adjust it. Here is how you do that:

```
# Adjust the configured number of replicas for a service
# Usage: docker service scale SERVICE=REPLICAS [SERVICE=REPLICAS...]
docker service scale web-service=4
```
[Read more about the Docker service create reference](https://docs.docker.com/engine/reference/commandline/service_create/)




