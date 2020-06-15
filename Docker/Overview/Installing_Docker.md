---
layout: default
title: Installing Docker 
parent: Docker Fundamental
nav_order: 2
---

# Installing Docker 

# Step 1 : Installing docker on Linux 
   - For Debian :
1. Install the packages necessary to add a new repository over HTTPS:
   ```
   sudo apt update
   sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg2
   ```
2. Import the repository’s GPG key using the following curl command:  
   ```
   curl -fsSL https://download.docker.com/linux/debian/gpg | sudo apt-key add -

   ```
On success, the command will return OK.

3. Add the stable Docker APT repository to your system’s software repository list:
```
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/debian $(lsb_release -cs) stable"
```
`$(lsb_release -cs)` will return the name of the Debian distribution. In this case, that is buster.
4. Update the apt package list and install the latest version of Docker CE (Community Edition):
```
$ sudo apt update
$ sudo apt install docker-ce
```
5. Once the installation is completed the Docker service will start automatically. To verify it type in:
```
$ sudo systemctl status docker

```
output 

```
   ● docker.service - Docker Application Container Engine
   Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
   Active: active (running) 
    Docs: https://docs.docker.com
```
6. At the time of writing, the latest stable version of Docker is 19.03.1:
```
$ docker -v

```
output
```
Docker version 19.03.1, build 74b1e89
```
# Executing the Docker Command Without Sudo

By default, only root and user with sudo privileges can execute Docker commands.

If you want to execute Docker commands without prepending sudo you’ll need to add your user to the docker group which is created during the installation of the Docker CE package. To do that, type in:
```
sudo usermod -aG docker $USER
```
$USER is an environment variable that holds your username.
Log out and log back in so that the group membership is refreshed.

Once done to verify that you can run docker commands without sudo type in:
```
docker container run hello-world
```
output:
```
Unable to find image 'hello-world:latest' locally
latest: Pulling from library/hello-world
0e03bdcc26d7: Pull complete 
Digest: sha256:d58e752213a51785838f9eed2b7a498ffa1cb3aa7f946dda11af39286c3db9a9
Status: Downloaded newer image for hello-world:latest

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
 https://docs.docker.com/get-started/
```
# For Ubuntu : Install Docker From a standard Ubuntu Repository  
1. Use the apt command to install the docker.io package:
```
sudo apt install docker.io
```
2. Start docker and enable it to start after the system reboot: 
```
sudo systemctl enable --now docker

```
3. Optionally give any user administrative privileges to docker: 
```
sudo usermod -aG docker SOMEUSERNAME
```
You will need to log out and log in to apply the changes. 
4. Check docker version: 
```
docker --version
```
5. Run docker test using the hello-world container: 
```
$ docker run hello-world

Hello from Docker!
This message shows that your installation appears to be working correctly.

To generate this message, Docker took the following steps:
 1. The Docker client contacted the Docker daemon.
 2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
    (amd64)
 3. The Docker daemon created a new container from that image which runs the
    executable that produces the output you are currently reading.
 4. The Docker daemon streamed that output to the Docker client, which sent it
    to your terminal.

To try something more ambitious, you can run an Ubuntu container with:
 $ docker run -it ubuntu bash

Share images, automate workflows, and more with a free Docker ID:
 https://hub.docker.com/

For more examples and ideas, visit:
https://docs.docker.com/get-started/

```
  
  
   

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

