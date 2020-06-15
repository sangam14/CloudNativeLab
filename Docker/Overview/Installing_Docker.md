---
layout: default
title: Installing Docker 
parent: Docker Fundamental
nav_order: 2
---

# Installing Docker 

# Step 1 : Installing docker on Linux 





# Lightweight Linux for Docker

Here’s a look at the lightweight, Linux-based operating systems that have sprung up recently to cater to Docker:

  - Atomic Host:- Built using components from the Red Hat side of the Linux universe, this operating system was one of the earlier lightweight GNU/Linux distributions to appear with a focus on containers. Because it’s tied to Red Hat, it supports Red Hat-friendly container components of the container stack, such as Kubernetes.
  - Alpine Linux:- This is the lightweight Linux distribution that Docker chose as the basis for packaging the Docker platform with a virtual machine so Windows users can easily start using Docker. Alpine, which dates back to 2005, is way, way older than Docker, however, and it is designed with more than Docker in mind.
   - CoreOS:-  Introduced in October 2013, when Docker was still in its infancy, CoreOS was designed from the beginning as a lightweight GNU/Linux distribution that could automate cluster workloads. That made it a good match for containerized environments when they began developing. Today, CoreOS is the operating system at the heart of the set of open-source container projects that CoreOS the company supports.
  - RancherOS:- This is Rancher’s solution for setting up a lightweight container server. Rancher itself is a holistic platform for building a container service, and RancherOS is not a requirement for running Rancher. But RancherOS is cool because it runs inside containers, making setup as simple as running a Docker container.


# Docker’s Killing the Linux Distribution

- With Docker, however, these considerations are much less important. When you run an application inside a container, the application always will run in the same basic way, no matter which flavor of Linux is hosting it.

- You also can run any container image on any type of Linux, and deploy it by following the same steps. That makes package managers and the size of package repositories unimportant.

- Plus, most Docker security tools, monitoring tools and orchestrators are pretty agnostic about the operating system that hosts them. Kubernetes is Kubernetes, whether your server is powered by Ubuntu, Red Hat or the distribution you built in your basement. Your monitoring and security tools most likely will run inside containers themselves, which makes the host flavor pretty irrelevant.

- So, if you migrate your application to Docker, most of the questions you previously had to ask yourself when deciding which type of Linux to run no longer are important.


# Step 2 : Installing Docker on Windows 


If you’ve ever tried to install Docker for Windows, you’ve probably came to realize that the installer won’t run on Windows 10 Home. Only Windows Pro, Enterprise or Education support Docker. Upgrading your Windows license is pricey, and also pointless, since you can still run Linux Containers on Windows without relying on Hyper-V technology, a requirement for Docker for Windows.




### Setting up your computer
Getting all the tooling setup on your computer can be a daunting task, but getting Docker up and running on your favorite OS has become very easy.

The *getting started* guide on Docker has detailed instructions for setting up Docker on [Mac](https://docs.docker.com/docker-for-mac/), [Linux](https://docs.docker.com/engine/installation/linux/) and [Windows](https://docs.docker.com/docker-for-windows/).

*If you're using Docker for Windows* make sure you have [shared your drive](https://docs.docker.com/docker-for-windows/#shared-drives).

*Important note* If you're using an older version of Windows or MacOS you may need to use [Docker Machine](https://docs.docker.com/machine/overview/) instead.

*All commands work in either bash or Powershell on Windows*

Once you are done installing Docker, test your Docker installation by running the following:
```
$ docker run hello-world
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
03f4658f8b78: Pull complete
a3ed95caeb02: Pull complete
Digest: sha256:8be990ef2aeb16dbcb9271ddfe2610fa6658d13f6dfb8bc72074cc1ca36966a7
Status: Downloaded newer image for hello-world:latest

Hello from Docker.
This message shows that your installation appears to be working correctly.
...
```

