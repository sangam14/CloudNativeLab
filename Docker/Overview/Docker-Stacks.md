---
layout: default
title: Docker Stacks
parent: Docker Fundamental
nav_order: 7
---

# Docker Stacks

- Docker stacks is the new and improved Docker Compose, and it is included in our installation. I bet you're thinking, Great. 
But what does that mean? What is the use case of Docker stacks? Great question! 

- Docker stacks is the way to leverage all of the functionality that the Docker commands, Docker images, Docker services, Docker volumes, Docker swarm,
and Docker networks, wrapping it all up in an easy-to-use, easy-to-understand, declarative document file that will instantiate and maintain a complex,
multi-image application on our behalf.

- Most of your work, which is still the easy part, will be in creating the compose file that will be used in the Docker stack commands. 
All of the really hard work will be done by Docker when it creates, starts, and manages all of the services required for your multi-service (multi-container) 
applications. All of this is handled by a single command on your part. Just like image, the container and swarm stacks are another Docker management group.
Let's take a look at the stack management commands:

```
$ docker stack --help 

Usage:  docker stack [OPTIONS] COMMAND

Manage Docker stacks

Options:
      --orchestrator string   Orchestrator to use (swarm|kubernetes|all)

Commands:
  deploy      Deploy a new stack or update an existing stack
  ls          List stacks
  ps          List the tasks in the stack
  rm          Remove one or more stacks
  services    List the services in the stack

Run 'docker stack COMMAND --help' for more information on a command.
[node1] (local) root@192.168.0.18 ~
$ 

```
- So, what do we have here? For all the power that this management group represents, it has a pretty simple set of commands.
The main command is the deploy command. It is the powerhouse! With this command (and a compose file), you will stand up your application, 
pulling any images that are not local to your environment, running the images, creating volumes as needed, creating networks as needed, 
deploying the defined number of replicas for each image, spreading them across your swarm for HA and load-balancing purposes, and more. 
This command is kind of like the one ring in The Lord of the Rings. In addition to deploying your application, 
you will use this same command to update running applications, when you need to do things such as scale your application.

- The next command in the management group is the list stacks command. As the name implies, the ls command allows you to get a list of all 
the stacks currently deployed to your swarm. When you need more detailed information about a particular stack that is running in your swarm, 
you will use the ps command to list all of the tasks of a particular stack. When it comes time to end of life a deployed stack, you will use 
the mighty rm command. And finally, rounding out the management commands, we have the services command, which allows us to get a list of the 
services that are part of the stack. There is one more important part of the stack puzzle, that being the `--orchestrator` option. With this option,
we can instruct Docker to use either Docker swarm or Kubernetes for the stack orchestration. Of course, to use Kubernetes, 
it must be installed, and to use swarm—which is the default if the option is not specified—swarm mode must be enabled.

- Check out the following links for more information:
    - Docker Compose Overview: https://docs.docker.com/compose/overview/
    - Docker stack command reference: https://docs.docker.com/engine/reference/commandline/stack/
    - Docker samples: https://github.com/dockersamples
    - Docker voting app example: https://github.com/dockersamples/example-voting-app
    - My fork of the voting app: https://github.com/EarlWaud/example-voting-app
    
    
# How to create and use a compose YAML files for Stacks

- he stack file is a YAML file, and is basically the same thing as a Docker Compose file. Both are YAML files that define a Docker base application. Technically, a stack file is a compose file that requires a specific version (or above) of the compose specification. Only the version 3.0 specification and above are supported by Docker stacks. If you have an existing project that uses Docker compose YAML files, and those files are using the version 2 or older specification, then you will need to update the YAML files to the version 3 spec to be able to use them with Docker stacks. It is worth noting that the same YAML file can be used with either Docker stacks or Docker compose (provided it is written using the version 3 specification or higher). 

- However, there are some instructions that will be ignored by one or the other tools. For example, the build instruction is ignored by Docker stacks. That is because one of the most significant differences between stacks and compose is that all utilized Docker images must be pre-created for use with stacks, whereas Docker images can be created as part of the process of standing up a compose-based application. Another significant difference is the stack file is able to define Docker services as part of the application.

Now would be a good time to clone the voting app project and the visualizer image repos:
```
# Clone the sample voting application and the visualizer repos
git clone https://github.com/EarlWaud/example-voting-app.git
git clone https://github.com/EarlWaud/docker-swarm-visualizer.git

```

Strictly speaking, you don't need to clone these two repos because all you really need is the stack compose file from the voting app. This is because all of the images are already created and publicly available to pull from hub.docker.com, and when you deploy the stack, the images will be pulled for you as part of the deployment. So, here is the command to obtain just the stack YAML file:

```
# Use curl to get the stack YAML file
curl -o docker-stack.yml\
    https://raw.githubusercontent.com/earlwaud/example-voting-app/master/docker-stack.yml

```

Of course, if you want to customize the app in any way, having the project local allows you to build your own versions of the Docker images and then deploy your custom version of the app using your custom images.

- Once you have the project (or at least the docker-stack.yml file) on your system, you can begin to play around with the Docker stack commands. So now, let's go ahead and kick things off by using the docker-stack.yml file to deploy our application. You will need to have your Docker nodes set up and have swarm mode enabled for this to work, so if you haven't done so already, set up your [swarm](https://containerlabs.kubedaily.com/Docker/Overview/Docker-swarm.html), Docker Swarm. Then, use the following command to deploy your example voting application:


```
[manager1] (local) root@192.168.0.15 ~
$ # Use curl to get the stack YAML file
$ curl -o docker-stack.yml\
>     https://raw.githubusercontent.com/earlwaud/example-voting-app/master/docker-stack.yml
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100  1692  100  1692    0     0  29684      0 --:--:-- --:--:-- --:--:-- 30214
[manager1] (local) root@192.168.0.15 ~
$ # Deploy the example voting application 
$ # using the downloaded stack YAML file
[manager1] (local) root@192.168.0.15 ~
$ docker stack deploy -c docker-stack.yml voteapp
Creating network voteapp_frontend
Creating network voteapp_backend
Creating network voteapp_default
Creating service voteapp_redis
Creating service voteapp_db
Creating service voteapp_vote
Creating service voteapp_result
Creating service voteapp_worker
Creating service voteapp_visualizer

```

- Let me quickly explaining this command: we are using the deploy command with the docker-stack.yml compose file, and naming our stack voteapp. This command will handle all of the configuration, deployment, and management for our new application. It will take some time to get everything up and running as defined in the `docker-stack.yml` file, so while that is happening, let's start diving into our stack compose file.

- By now, you know we are using the docker-stack.yml file. So, as we explain the various parts of the stack compose file, you can bring that file up in your favorite editor, and follow along. Here we go!

- The first thing we are going to look at is the top-level keys. In this case, they are as follows:
   -  version
   -  services
   -  networks
   -  volumes
   
As mentioned previously, the version must be at least 3 to work with Docker stacks. Looking at line 1 (the version key is always on line 1) in the docker-stack.yml file, we see the following: 

```
version : "3" 
service : ...

networks: 
  frontend :
  backend :
  
volumes:
  dbdata :

```

Perfect! We have a compose file that is at the version 3 specification. Skipping over the (collapsed) services key section for a minute, let's take a look at the networks key and then the volumes key. In the networks key section, we are instructing Docker to create two networks, one named frontend, and one named backend. Actually, in our case, the networks will have the names voteapp_frontend and voteapp_backend. This is because we named our stack voteapp, and Docker will prepend the name of the stack to the various components it deploys as part of the stack. Simply by including the names for our desired networks within the networks key of our stack file, Docker will create our networks when we deploy our stack. We can provide specific details for each network but if we don't provide any, then certain default values will be used. It's probably been long enough for our stack to deploy our networks, so let's use the network list command and take a look at what networks we have now:

```
$ docker network ls
NETWORK ID          NAME                DRIVER              SCOPE
530ece081993        bridge              bridge              local
67febb61eb13        docker_gwbridge     bridge              local
f2e0b7b50a74        host                host                local
g4fdkof1mktn        ingress             overlay             swarm
258ea024a210        none                null                local
pft6syk67nv5        voteapp_backend     overlay             swarm
kh63cczry3un        voteapp_default     overlay             swarm
pefcgikbakr9        voteapp_frontend    overlay             swarm

```

- There they are: `voteapp_frontend` and `voteapp_backend`. You might be wondering what the voteapp_default network is. When you deploy a stack, you will always get a default swarm network and all containers are attached to it if they don't have any other network connection defined for them in the stack compose file. This is very cool, right?! You didn't have to do any docker network create commands, and your desired networks are created and ready to use in your application.

- The volumes key section does pretty much the same thing as the networks key section, except it does it for volumes. You get your defined volumes created automatically when you deploy the stack. The volumes are created with default settings if no additional configuration is provided in the stack file. In our example, we are asking Docker to create a volume named db-data. As you may have guessed, the volume created actually has the name of `voteapp_db-data` because Docker prepended the name of our stack to the volume name. In our case, it looks like this:

```
$ docker volume ls
DRIVER              VOLUME NAME
local               voteapp_db-data

```
- So, deploying our stack created our desired networks and our desired volume. All with the easy-to-create, and easy-to-read-and-understand content in our stack compose file. OK, so we now have a good grasp of three of the four top-level key sections in our stack compose file. Now, let's return to the services key section. If we expand this key section, we will see definitions for each of the services we wish to deploy as part of the application. In the case of the docker-stack.yml file, we have six services defined. These are redis, db, vote, result, worker, and visualizer. In the stack compose file, they look like this:

```
 service: ...
      redis: ...
         db:  ...
       vote:  ...
     result:  ...
     worker:  ...
virtualizer:  ...

```
Let's expand the first one, redis, and take a closer look at what is defined as the redis service for our application:

```
version: "3"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure


```

- Let's examine the keys in the redis service now. First up, we have the image key. The image key is required for the service definition. This key is telling docker that the Docker image to pull and run for this service is redis:alpine. As you should understand now, this means that we are using the official redis image from hub.docker.com, requesting the version tagged as alpine.

- The next key, ports, is defining what port the images will be exposing from the container, and from the hosts. In this case, the port on the host that is to be mapped to the container's exposed port (6379) is left to Docker to assign. You can find the port assigned using the `docker container ls` command. In my case, the redis service is mapping port `30000` on the host to port `6379` on the container. The next key used is networks.

- We already have seen that deploying the stack will create our networks for us. This directive is telling Docker which networks that the redis replica containers should be connected to; in this case it is the frontend network. If we inspect a redis replica container, examining the networks section, we will see this to be accurate. You can have a look at your deployment with a command such as this (note that the container name will be slightly different on your system):

```
# Inspect a redis replica container looking at the networks
docker container inspect voteapp_redis.1.nwy14um7ik0t7ul0j5t3aztu5  \
      --format '{{json .NetworkSettings.Networks}}' | jq


```

- In our example, you should see that the container is attached to two networks: the ingress network and our `voteapp_frontend` network.

- The next key in our redis service definition is the deploy key. This is a key category that was added to the compose file specification with version 3. It is what defines the specifics for running the containers based on the image in this service: in this case, the redis image. It is essentially the orchestration instructions. The replicas tag tells docker how many copies or containers should be running when the application is fully deployed. 

- In our example, we are stating that we only need one instance of the redis container running for our application. The update_config key provides two sub keys, parallelism and delay, that tell Docker how many container replicas should be started in parallel, and how much time to wait between starting each parallel set of container replicas. Of course, with one replica, the parallelism and delay details have little use. If the value for replicas were something greater, such as 10, our update_config keys would result in two replicas starting at a time, with a wait of 10 seconds between starts. The final deploy key is `restart_policy`, and this defines the conditions that a new replica will be created in a deployed stack. In this case, if a redis container fails, a new redis container will be started to take its place. Let's take a look at the next service in our application, the db service:

```
db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

```

- The db service will have several keys in common with the redis service, but with different values. First, we have the image key. This time we are indicating that we want the official postgres image with the tag for version 9.4. Our next key is the volumes key. We are indicating that we are using the volume named db-data, and that in the DB container the volume should be mounted at `/var/lib/postgresql/data`. Let's take a look at the volume information in our environment:

```
$ docker volume inspect voteapp_db-data 
[
    {
        "CreatedAt": "2020-08-26T17:55:22Z",
        "Driver": "local",
        "Labels": {
            "com.docker.stack.namespace": "voteapp"
        },
        "Mountpoint": "/var/lib/docker/volumes/voteapp_db-data/_data",
        "Name": "voteapp_db-data",
        "Options": null,
        "Scope": "local"
    }
]


```

```
vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - 5000:80
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

```

- You should be starting to get familiar with these keys and their values. Here in the vote service we see that the image defined is not one of the official container images, but instead is in a public repo named dockersamples. Within that repo, we are using the image named examplevotingapp_vote, with a version tag of before. Our ports key is telling Docker, and us, that we want to open port 5000 on the swarm hosts and have traffic on that port mapped to port 80 in the running vote service containers. As it turns out, the vote service is the face of our application and we will access it via port 5000. Since it is a service, we can access it by going to port 5000 on any of the hosts in the swarm, even when a particular host is not running one of the replicas.

- Looking at the next key, we see that we are attaching the frontend network to our vote service containers. Nothing new there, however, as our next key is one we have not seen before: the depends_on key. This key is telling Docker that our vote service requires the redis service to function. What this means to our deploy command is that the service or services that are depended on need to be started before starting this service. Specifically, the redis service needs to be started before the vote service. One key distinction here is that I said started. This does not mean that the depended-upon service has to be running before starting this service; the depended-on service just has to be started before it. Again, specifically, the redis service does not have to be at the state of running before starting the vote service, it just has to be started before the vote service is started. There is nothing we haven't seen yet in the deploy key in for the vote service, with the only difference being that we are asking for two replicas for the vote service. Are you beginning to understand the simplicity and the power of the service definition in the stack compose file?

- The next service defined in our stack compose file is for the result service. However, since there are no keys present in that service definition that we haven't seen in the previous services, I will skip the discussion on the result service, and move on to the worker service where we'll see some new stuff. Here is the worker service definition:

```
worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]


```

- You know about the image key and what it means. You know about the networks key and what it means too. You know about the deploy key, but we have some new sub-keys here so let's talk about them, starting with the mode key. You may recall from our discussion of services in Chapter 5, Docker Swarm, that there is a --mode parameter that can have one of two values: global or replicated. This key is exactly the same as the parameter we saw in Docker Swarm. The default value is replicated, and so if you do not specify the mode key, you will get the replicated behavior, which is to have exactly the number of replicas that are defined (or one replica if no number of replicas is specified). Using the other value option of global will ignore the replicas key and deploy exactly one container to every host in the swarm.


- The next key that we have not seen before in this stack compose file is the labels key. The location of this key is significant as it can appear as its own upper-level key, or as a sub-key to the deploy key. What is the distinction? When you use the labels key as a sub-key to the deploy key, the label will be set only on the service. When you use the labels key as its own upper-level key, the label will be added to each replica, or container, deployed as part of the service. In our example, the APP=VOTING label will be applied to the service because the labels key is a sub-key to the deploy key. Again, let's see this in our environment:

```
# Inspect the worker service to see its labels
docker service inspect voteapp_worker \
 --format '{{json .Spec.Labels}}' | jq
```
Here is what that looks like on my system:
```
$ # Inspect the worker service to see its labels
$ docker service inspect voteapp_worker \
>  --format '{{json .Spec.Labels}}' | jq
{
  "APP": "VOTING",
  "com.docker.stack.image": "dockersamples/examplevotingapp_worker",
  "com.docker.stack.namespace": "voteapp"
}
[manager1] (local) root@192.168.0.15 ~
$ 

```

Executing an inspect command on a worker container to view the labels on it will show that the APP=VOTING label does not appear. If you want to confirm this on your system, the command will look like this (with a different container name):

```
docker container inspect voteapp_worker.1.3sz2odsnr7d9d3x1ai2tdji29 \
>      -f '{{json .Config.Labels}}' | jq
{
  "com.docker.stack.namespace": "voteapp",
  "com.docker.swarm.node.id": "20lqc0y6n4zcrycwt3m6zqzey",
  "com.docker.swarm.service.id": "z3qx4rizc6je670ul2sa1ozvf",
  "com.docker.swarm.service.name": "voteapp_worker",
  "com.docker.swarm.task": "",
  "com.docker.swarm.task.id": "3sz2odsnr7d9d3x1ai2tdji29",
  "com.docker.swarm.task.name": "voteapp_worker.1.3sz2odsnr7d9d3x1ai2tdji29"
}


```

Two new sub-keys for the restart_policy key are the max_attempts and window keys. You can probably guess their purpose; the max_attempts key tells Docker to keep trying to start the worker containers if they fail to start, up to three times before giving up. The window key tells Docker how long to wait before retrying to start a worker container if it failed to start previously. Pretty straightforward, right? Again, these definitions are easy to set up, easy to understand, and extremely powerful for orchestrating the services of our application.

Alright. We have one more service definition to review for new stuff, that being the visualizer service. Here is what it looks like in our stack compose file:

```

 visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]



```

The only truly new key is the stop_grace_period key. This key tells Docker how long to wait after it tells a container to stop before it will forcefully stop the container. The default time period, if the stop_grace_period key is not used, is 10 seconds. When you need to update a stack, essentially do a re-stack, the containers of a service will be told to shut down gracefully. Docker will wait for the amount of time specified in the stop_grace_period key, or for 10 seconds if the key is not provided. If the container shuts down during that time, the container will be removed, and a new container will be started in its place. If the container does not shut down during that window of time, it will be stopped by force, killing it, then removing it, then starting a new container to take its place. The significance of this key is that it allows the necessary time for containers that are running processes that take longer to stop gracefully to actually stop gracefully.


# The rest of the stack commands


Now, let's take a quick look at our other stack-related commands through the lens of the swarm where we deployed our voteapp stack. First up, we have the list stacks command: docker stack ls. Giving that a try looks like this:

```
# List the stacks deployed in a swarm
docker stack ls

```


Here is what it looks like in the example environment:
```
$ # List the stacks deployed in a swarm
$ docker stack ls
NAME                SERVICES            ORCHESTRATOR
voteapp             6                   Swarm


```
This is showing that we have one stack named voteapp currently deployed, and that it is composed of six services and is using swarm mode for its orchestration. Knowing the name of a deploy stack allows us to gather more information about it using the other stack commands. Next up is the list stack tasks command. Let's give this command a try in our example environment:

```
# List the tasks for our voteapp stack filtered by desried state
docker stack ps voteapp --filter desired-state=running


```
Here are the results in my environment right now; yours should look very similar:

```
[manager1] (local) root@192.168.0.15 ~
$ docker stack ps voteapp --filter desired-state=running
ID                  NAME                   IMAGE                                          NODE                DESIRED STATE       CURRENT STATE            ERROR               PORTS
zcsgldt6p6ec        voteapp_result.1       dockersamples/examplevotingapp_result:before   manager1            Running             Running 11 minutes ago                       
si13cdpcrizt        voteapp_visualizer.1   dockersamples/visualizer:stable                manager2            Running             Running 3 hours ago                          
kg1szl258kus        voteapp_vote.1         dockersamples/examplevotingapp_vote:before     worker1             Running             Running 3 hours ago                          
vcy6b3rwyqec        voteapp_redis.1        redis:alpine                                   manager3            Running             Running 3 hours ago                          
qvzz2oddsqcm        voteapp_vote.2         dockersamples/examplevotingapp_vote:before     worker2             Running             Running 3 hours ago    


```
Now, we will have a look at the stack services command. This command will give us a nice summary of the services that are deployed as part of our stack application. The command looks like this:

```
$ # Look at the services associated with a deployed stack
$ docker stack services voteapp
ID                  NAME                 MODE                REPLICAS            IMAGE                                          PORTS
4fqgkj8el60m        voteapp_result       replicated          1/1                 dockersamples/examplevotingapp_result:before   *:5001->80/tcp
ie4ezdkif73b        voteapp_vote         replicated          2/2                 dockersamples/examplevotingapp_vote:before     *:5000->80/tcp
j93k6f2sa6sv        voteapp_db           replicated          0/1                 postgres:9.4                                   
k4771o8421e0        voteapp_visualizer   replicated          1/1                 dockersamples/visualizer:stable                *:8080->8080/tcp
lbx83i1xpfmv        voteapp_redis        replicated          1/1                 redis:alpine                                   *:30000->6379/tcp
z3qx4rizc6je        voteapp_worker       replicated          0/1                 dockersamples/examplevotingapp_worker:latest  

```

command provides some very useful information. We can quickly see the names of our services, the number of replicas desired, and the actual number of replicas for each service. We can see the image used to deploy each service, and we can see the port mapping used for each service. Here, we can see the visualizer service is using port 8080, as we mentioned earlier. We can also see that our vote service is exposed on port 5000 of our swarm hosts. Let's have a look at what we are presenting in our voteapp by browsing to port 5000 

- There is one final stack command: the remove command. We can quickly and easily take down an application deployed with the stack deploy command by issuing the rm command. Here is what that looks like:


```
# Remove a deploy stack using the rm command
docker stack rm voteapp


```
Now you see it, now you don't:

```
$ # Remove a deploy stack using the rm command
[manager1] (local) root@192.168.0.15 ~
$ docker stack rm voteapp
Removing service voteapp_db
Removing service voteapp_redis
Removing service voteapp_result
Removing service voteapp_visualizer
Removing service voteapp_vote
Removing service voteapp_worker
Removing network voteapp_default
Removing network voteapp_frontend
Removing network voteapp_backend
[manager1] (local) root@192.168.0.15 ~
$ 
```

- Check out the following links for more information:

   - [The compose file reference](https://docs.docker.com/compose/compose-file/)
   - [Some compose file examples](https://github.com/play-with-docker/stacks)
   - [Docker sample images on Docker hub](https://hub.docker.com/u/dockersamples/)
   - [Official redis image tags found on Docker hub](https://hub.docker.com/r/library/redis/tags/)
   - [A great article about using the Docker daemon socket](https://medium.com/lucjuggery/about-var-run-docker-sock-3bfd276e12fd)
   - [The stack deploy command reference](https://docs.docker.com/engine/reference/commandline/stack_deploy/)
   - [The stack ps command reference](https://docs.docker.com/engine/reference/commandline/stack_ps/)
   - [The stack services command reference](https://docs.docker.com/engine/reference/commandline/stack_services/)
