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
    
    
    



