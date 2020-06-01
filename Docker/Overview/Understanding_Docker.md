---
layout: default
title: Understanding Docker 
parent: Docker Fundamental
nav_order: 1
---
# Understanding Docker 
![img](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/four-components-of-computer-system.png) 
<br>

For Understanding Docker see the concept of operating system first :
{: .label .label-blue } 

1.Hardware – provides basic computing resources - CPU, memory, I/O devices
{: .label .label-blue }
2.Operating system Controls and coordinates use of hardware among various applications and users
{: .label .label-blue }
3.Application programs – define the ways in which the system resources are used to solve the computing problems of the users Word processors, compilers, web browsers, database systems, video games
{: .label .label-blue }
4.Users- People, machines, other computers
{: .label .label-blue }

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

|  Pros:       | Cons      | 
|:-------------|:------------------|
| Monolithic applications are very simple to develop because of all the tools and IDEs support to that kind of application by default. |Very difficult to understand and create the patches for monolithic applications.|
|Very easy to deploy because all components are packed into one bundle.| Adapting to new technology is very challengeable.|
|Easy to scale the whole application|Very difficult to maintain CI/CD pipeline|
| |Maintainability is very high and braking of one code line will stop the whole process| 
| |Take a long time to startup because all the components need to get started | 
| |One component failure will cause the whole system to fail | 








