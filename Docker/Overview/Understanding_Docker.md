---
layout: default
title: Understanding Docker 
parent: Docker Fundamental
nav_order: 1
---
# Understanding Docker 
![img](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/four-components-of-computer-system.png) 
<br>

For Understanding Docker see the concept of operating system first : <br>


1.Hardware – provides basic computing resources - CPU, memory, I/O devices. <br>

2.Operating system Controls and coordinates use of hardware among various applications and users<br>

3.Application programs – define the ways in which the system resources are used to solve the computing problems of the users Word processors, compilers, web browsers, database systems, video games. <br>

4.Users- People, machines, other computers. <br>


we have above 4 component that all you need to run any application .. as moving forward from physical Computaion world we have moved forward to virtualization technology, but Why ?

lets take simple anology where you have one host and top of that host you can run limited application and resource and infra cost is to high to mantain bigger application and thats where we moved to virtualization world . 


# Understanding Virtualization
![img](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/Virtualization.png)

Virtualization is technology that lets you create useful IT services using resources that are traditionally bound to hardware. It allows you to use a physical machine’s full capacity by distributing its capabilities among many users or environments.
![img](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/virtualizationvscontainerlization.png)
VMs, however, can take up a lot of system resources. Each VM runs not just a full copy of an operating system, but a virtual copy of all the hardware that the operating system needs to run. This quickly adds up to a lot of RAM and CPU cycles. That’s still economical compared to running separate actual computers, but for some applications it can be overkill, which led to the development of containers. check series [ Birth of Containerlization](http://containerlabs.kubedaily.com/Birth_of_Containerization/README.html){: .btn .btn-green .mr-4 }for more details 
# What’s the Diff: VMs vs Containers

| Virtual Machine       | Containers       | 
|:-------------|:------------------|
| Heavyweight  |   Lightweight     | 
| Limited performance |   Native performance | 
| Each VM runs in its own OS   |   All containers share the host OS  | 
| Hardware-level virtualization   | OS virtualization |
| Startup time in minutes | Startup time in milliseconds |
| Allocates required memory | Requires less memory space |
| Fully isolated and hence more secure | Process-level isolation, possibly less secure |

# Why We Are Moving From Monolithic To Microservices   
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/monolith.png)
This kind of application is not responsible for a single task, but they need several tasks to complete a particular responsibility. In monolithic applications, all the services are bundled into one package and run as one process. The user interface, data access layer, and data store layers are tightly coupled in monolithic applications. Usually, large teams work with monolithic applications, and they are not suitable for container-based deployments.

|  Pros:       | Cons              | 
|:-------------|:------------------|
| Monolithic applications are very simple to develop because of all the tools and IDEs support to that kind of application by default. |Very difficult to understand and create the patches for monolithic applications.|
|Very easy to deploy because all components are packed into one bundle.| Adapting to new technology is very challengeable.|
|Easy to scale the whole application|Very difficult to maintain CI/CD pipeline|
| |Maintainability is very high and braking of one code line will stop the whole process| 
| |Take a long time to startup because all the components need to get started | 
| |One component failure will cause the whole system to fail | 

<br>

# Key Concepts of Microservice Architecture

Before you start building your own applications using microservices, you need to be clear about the scope and functionalities of your application.

Following are some guidelines to be followed while discussing microservices.

   -  As a developer, when you decide to build an application separate the domains and be clear with the functionalities.
   -  Each microservice you design shall concentrate only on one service of the application.
   -  Ensure that you have designed the application in such a way that each service is individually deployable.
   -  Make sure that the communication between microservices is done via a stateless server.
   -  Each service can be furthered refactored into smaller services, having their own microservices.


# Problem Statement 

While Uber started expanding worldwide this kind of framework introduced various challenges. The following are some of the prominent challenges

- All the features had to be re-built, deployed and tested again and again to update a single feature.
- Fixing bugs became extremely difficult in a single repository as developers had to change the code again and again.
- Scaling the features simultaneously with the introduction of new features worldwide was quite tough to be handled together.

# Solution

- To avoid such problems Uber decided to change its architecture and follow the other hyper-growth companies like Amazon, Netflix, Twitter and many others. Thus, Uber decided to break its monolithic architecture into multiple codebases to form a microservice architecture.
Refer to the diagram below to look at Uber microservice architecture

![img](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/microservice.png)
The major change that we observe here is the introduction of API Gateway through which all the drivers and passengers are connected. From the API Gateway, all the internal points are connected such as passenger management, driver management, trip management and others.
   -  The units are individual separate deployable units performing separate functionalities.
       -  For Example: If you want to change anything in the billing microservices, then you just have to deploy only billing microservices and don’t have to deploy the others.
   -  All the features were now scaled individually i.e. The interdependency between each and every feature was removed.
      - For Example, we all know that the number of people searching for cabs is more comparatively more than the people actually booking a cab and making payments. This gets us an inference that the number of processes working on the passenger management microservice is more than the number of processes working on payments.

In this way, Uber benefited by shifting its architecture from monolithic to microservices.


# Containers are just another option
- Virtual Machines
     - Each virtual machine (VM) includes the app, the necessary binaries and libraries and an entire guest operating system 
- Containers
   - Containers include the app & all of its dependencies, but share the kernel with other containers.
   - Run as an isolated process in userspaceon the hostOS
   - Not tied to any specific infrastructure – containers run on any computer, infrastructure and cloud.

# Docker Architecture

![img](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/DockerAchitecture.png)
Docker Host:- A Docker host is a physical computer system or virtual machine running Linux. This can be your laptop, server or virtual machine in your data center, or computing resource provided by a cloud provider.

The component on the host that does the work of building and running containers is the Docker Daemon. The daemon starts each container using a template for the container’s specific runtime environment called an image, which is retrieved from an image repository, like the public repositories hosted at Docker Hub. (You can search for image repositories using the official Docker registry.)

# Docker Client / Docker Daemon / images / Container 
![img](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/Docker_Daemon.png)

- Images might sound like guest operating systems for virtual machines, but they are much more lightweight and much more efficient. They rely on the shared Linux kernel of the host, so they only add the binaries and other resources needed by the application, and they don’t go through a virtualization layer for execution.

- Images are actually composed of layers of images that are cached on the host and shared across containers, and they leverage the copy-on-write union file system for storage efficiency and performance (because the image doesn’t need to be copied for each new container, and unlike a guest OS, the image doesn’t need to boot since the shared kernel is already running).

- You manage images and containers by communicating with the Docker Daemon from a client via the Docker REST API. For interactive sessions, the Docker command line interface (CLI) provides a convenient wrapper for the API. By default, the daemon can only accept local connections over a Unix domain socket.

- For a computer to be a Docker host, it needs to be running Linux. If the computer (such as your laptop) happens to be running another operating system (like OS X and Windows), then it will be necessary to use a virtual machine that runs Linux.








