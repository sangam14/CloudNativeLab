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
# For RHEL 8 / CentOS 8 

- The latest release of the RHEL 8 / CentOS 8. Red Hat has built its own tools, buildah and podman, which aim to be compatible with existing docker images and work without relying on a daemon, allowing the creation of containers as normal users, without the need of special permissions (with some limitations: e.g. at the moment of writing, it's still not possible to map host ports to the container without privileges).

- Some specific tools, however, are still missing: an equivalent of docker-compose, for example does not exists yet. In this tutorial we will see how to install and run the original Docker CE on Rhel8 by using the official Docker repository for CentOS7.

1.  Adding the external repository
- Since Docker is not available on RHEL 8 / CentOS 8, we need to add an external repository to obtain the software. In this case we will use the official Docker CE CentOS repository: this is, at the moment of writing, the only way to install Docker CE on RHEL 8 / CentOS 8.

- The `dnf config-manager` utility let us, among the other things, easily enable or disable a repository in our distribution. By default, only the `appstream `and `baseos` repositories are enabled on Rhel8; we need to add and enable also the `docker-ce `repo. All we need to do to accomplish this task, is to run the following command:

```
$ sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

```

2. We can verify that the repository has been enabled, by looking at the output of the following command:

```
$ sudo dnf repolist -v

```
The command above will return detailed information about all the enabled repositories. This is what you should see at this point:

```
Repo-id      : docker-ce-stable
Repo-name    : Docker CE Stable - x86_64
Repo-revision: 1549905809
Repo-updated : Mon 11 Feb 2019 06:23:29 PM CET
Repo-pkgs    : 30
Repo-size    : 618 M
Repo-baseurl : https://download.docker.com/linux/centos/7/x86_64/stable
Repo-expire  : 172,800 second(s) (last: Mon 18 Feb 2019 10:23:54 AM CET)
Repo-filename: /etc/yum.repos.d/docker-ce.repo

Repo-id      : rhel-8-for-x86_64-appstream-rpms
Repo-name    : Red Hat Enterprise Linux 8 for x86_64 - AppStream Beta (RPMs)
Repo-revision: 1542158694
Repo-updated : Wed 14 Nov 2018 02:24:54 AM CET
Repo-pkgs    : 4,594
Repo-size    : 4.9 G
Repo-baseurl : https://cdn.redhat.com/content/beta/rhel8/8/x86_64/appstream/os
Repo-expire  : 86,400 second(s) (last: Mon 18 Feb 2019 10:23:55 AM CET)
Repo-filename: /etc/yum.repos.d/redhat.repo

Repo-id      : rhel-8-for-x86_64-baseos-rpms
Repo-name    : Red Hat Enterprise Linux 8 for x86_64 - BaseOS Beta (RPMs)
Repo-revision: 1542158719
Repo-updated : Wed 14 Nov 2018 02:25:19 AM CET
Repo-pkgs    : 1,686
Repo-size    : 925 M
Repo-baseurl : https://cdn.redhat.com/content/beta/rhel8/8/x86_64/baseos/os
Repo-expire  : 86,400 second(s) (last: Mon 18 Feb 2019 10:23:56 AM CET)
Repo-filename: /etc/yum.repos.d/redhat.repo
Total packages: 6,310
```

# Installing docker-ce

The `docker-ce-stable` repository is now enabled on our system. The repository contains several versions of the docker-ce package, to display all of them, we can run:

```
$ dnf list docker-ce --showduplicates | sort -r
docker-ce.x86_64            3:19.03.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:19.03.0-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.9-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.8-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.7-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.6-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.5-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.4-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.3-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.2-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.1-3.el7                     docker-ce-stable
docker-ce.x86_64            3:18.09.0-3.el7                     docker-ce-stable
docker-ce.x86_64            18.06.3.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.2.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.1.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.06.0.ce-3.el7                    docker-ce-stable
docker-ce.x86_64            18.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            18.03.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.12.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.09.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.06.0.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.3.ce-1.el7                    docker-ce-stable
docker-ce.x86_64            17.03.2.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.1.ce-1.el7.centos             docker-ce-stable
docker-ce.x86_64            17.03.0.ce-1.el7.centos             docker-ce-stable

```
 However, if you still want to proceed with the installation, here are the possible methods that can be used to avoid the dependencies issues:

  - Install a specific version of docker-ce which requires an installable version of the containerd.io package;
  - Force the installation providing the --nobest option
  - Install the latest available containerd.io rpm manually;

# Install a specific version of docker-ce

```
$ sudo dnf install docker-ce-3:18.09.1-3.el7
```
- Force the installation of docker-ce with the --nobest option:

    - Normally, when installing a package, the best available candidate is selected from a repository. In this case, for example, the installation of the latest version of `docker-ce `is attempted (and fails). By using the `--nobest` option, we can change this behavior so that the first version of `docker-ce` with satisfiable dependencies is selected as "fallback", in this case `3:18.09.1-3.el7`
    
```
$ sudo dnf install --nobest docker-ce
Dependencies resolved.

Problem: package docker-ce-3:19.03.2-3.el7.x86_64 requires containerd.io >= 1.2.2-3, but none of the providers can be installed
  - cannot install the best candidate for the job
  - package containerd.io-1.2.2-3.3.el7.x86_64 is excluded
  - package containerd.io-1.2.2-3.el7.x86_64 is excluded
  - package containerd.io-1.2.4-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.5-3.1.el7.x86_64 is excluded
  - package containerd.io-1.2.6-3.3.el7.x86_64 is excluded
=======================================================================================================================================================
 Package                            Arch         Version                                                  Repository                              Size
=======================================================================================================================================================
Installing:
 docker-ce                          x86_64       3:18.09.1-3.el7                                          docker-ce-stable                        19 M
Installing dependencies:
 containerd.io                      x86_64       1.2.0-3.el7                                              docker-ce-stable                        22 M
 docker-ce-cli                      x86_64       1:19.03.2-3.el7                                          docker-ce-stable                        39 M
 container-selinux                  noarch       2:2.94-1.git1e99f1d.module+el8.0.0+4017+bbba319f         rhel-8-for-x86_64-appstream-rpms        43 k
 tar                                x86_64       2:1.30-4.el8                                             rhel-8-for-x86_64-baseos-rpms          838 k
 libcgroup                          x86_64       0.41-19.el8                                              rhel-8-for-x86_64-baseos-rpms           70 k
 python3-policycoreutils            noarch       2.8-16.1.el8                                             rhel-8-for-x86_64-baseos-rpms          2.2 M
 python3-libsemanage                x86_64       2.8-5.el8                                                rhel-8-for-x86_64-baseos-rpms          127 k
 python3-setools                    x86_64       4.2.0-2.el8                                              rhel-8-for-x86_64-baseos-rpms          598 k
 checkpolicy                        x86_64       2.8-2.el8                                                rhel-8-for-x86_64-baseos-rpms          338 k
 python3-audit                      x86_64       3.0-0.10.20180831git0047a6c.el8                          rhel-8-for-x86_64-baseos-rpms           85 k
 policycoreutils-python-utils       noarch       2.8-16.1.el8                                             rhel-8-for-x86_64-baseos-rpms          228 k
Skipping packages with broken dependencies:
 docker-ce                          x86_64       3:19.03.2-3.el7                                          docker-ce-stable                        24 M

Transaction Summary
=======================================================================================================================================================
Install  12 Packages
Skip      1 Package

Total download size: 85 M
Installed size: 351 M
Is this ok [y/N]: 

```
# Install the latest available containerd.io package manually

if we stricly need to install the latest version of docker-ce, we can install the required version of containerd.io manually, by running:

```
$ sudo dnf install https://download.docker.com/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
```

After the package is installed, we can simply install the latest docker-ce:

```
$ sudo dnf install docker-ce
Dependencies resolved.
=======================================================================================================================================================
  Package                          Arch                      Version                             Repository                                        Size
=======================================================================================================================================================
Installing:
  docker-ce                        x86_64                    3:19.03.2-3.el7                     docker-ce-stable                                  24 M
Installing dependencies:
  docker-ce-cli                    x86_64                    1:19.03.2-3.el7                     docker-ce-stable                                  39 M
  tar                              x86_64                    2:1.30-4.el8                        rhel-8-for-x86_64-baseos-rpms                    838 k
  libcgroup                        x86_64                    0.41-19.el8                         rhel-8-for-x86_64-baseos-rpms                     70 k

Transaction Summary
=======================================================================================================================================================
Install  4 Packages

Total download size: 65 M
Installed size: 275 M
Is this ok [y/N]:

```

This option is less convenient since the containerd.io package is not installed as a dependency of docker-ce, therefore it will not be removed automatically when the latter is uninstalled from the system.

Whatever method we use to install docker-ce, as said before, in order to make DNS resolution work inside Docker containers, we must disable firewalld (a system reboot may be also needed):

```
$ sudo systemctl disable firewalld

```

# Start and enable the docker daemon

Once docker-ce is installed, we must start and enable the docker daemon, so that it will be also launched automatically at boot. The command we need to run is the following:

```
$ sudo systemctl enable --now docker

```

At this point, we can confirm that the daemon is active by running:

```
$ systemctl is-active docker
active

```

Similarly, we can check that it is enabled at boot, by running:

```
$ systemctl is-enabled docker
enabled
```

# Global installation - Docker Compose

The way we should install docker-compose varies depending on whether we want to install it globally or just for a single user. At the moment of writing, the only way to install it globally is to download the binary from the github page of the project:

```
$ curl -L "https://github.com/docker/compose/releases/download/1.23.2/docker-compose-$(uname -s)-$(uname -m)" -o docker-compose
```
Once the binary is downloaded, we move it into /usr/local/bin and we make it executable:

```
$ sudo mv docker-compose /usr/local/bin && sudo chmod +x /usr/local/bin/docker-compose
```

The `/usr/local` hierarchy is not chosen randomly. This directory structure is made to be used for files installed by the local administrator manually (for software compiled from source, for example), in order to ensure separation from the software installed with the system package manager.

Although it's possible for a normal user to run docker-related commands if he is part of the docker group (the group is automatically created when we install docker-ce), by default they must be executed with root privileges for security reasons. When we need to do the latter, since the `/usr/local/bin` directory is not in the root user's PATH, we either need to call the binary specifying its location or add `/usr/local/bin` to the PATH itself. The first option is the one which I recommend in this case.

# Per-user installation

If our user is part of the docker group, and thus it is allowed to run docker commands, and since docker-compose is available as a python package, we can also install it using pip, the python package manager. First, make sure pip itself is installed:

```
$ sudo dnf install python3-pip
```
To obtain docker-compose we run:

```
$ pip3.6 install docker-compose --user
```
Please notice that even if would be possible to run pip as root to install a package globally, this is not recommended and highly discouraged

# Testing docker

We installed docker and docker-compose, now to check that everything works as expected, we can try to build an image and run a container: in this case we will use the official httpd one. All we have to do is to launch the following command:

```
sudo docker run --rm --name=linuxconfig-test -p 80:80 http
``` 
Since the httpd image does not exists locally it will be automatically fetched and built. Finally, a container based on it will be launched in the foreground (it will be automatically removed when stopped). We should be able to see the It works! message when we reach our machine ip via browser.

# Conclusions

Red Hat Enterprise Linux 8 does not support Docker: on this distribution it has been replaced by Red Hat own tools like buildah and podman, which are compatible with Docker but don't need a server/client architecture to run. Using native tools, where possible, is always the recommended way to go, but for a reason or another you may still want to install the original Docker. In this tutorial, we saw how it is possible to install Docker CE on Rhel8, by using the official Docker repository for CentOS7, which is a 100% compatible clone.

This is not an ideal solution, and as we saw, at the moment, some workarounds are needed to make Docker work on RHEL8. If some new issues arises, or better solutions to the problems mentioned above are found, this article will be updated accordingly. Stay tuned.

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

