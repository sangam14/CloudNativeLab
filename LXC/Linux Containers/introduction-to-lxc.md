# Introduction to Linux Containers

Nowadays, deploying applications inside some sort of a Linux container is a widely adopted practice, primarily due to the evolution of the tooling and the 
ease of use it presents. Even though Linux containers, or operating-system-level virtualization, in one form or another, have been around for more than a decade,
it took some time for the technology to mature and enter mainstream operation. One of the reasons for this is the fact that hypervisor-based technologies such as 
KVM and Xen were able to solve most of the limitations of the Linux kernel during that period and the overhead it presented was not considered an issue. However, 
with the advent of kernel namespaces and control groups (cgroups) the notion of a light-weight virtualization became possible through the use of containers.


- I'll cover the following topics:

   - Evolution of the OS kernel and its early limitations

   - Differences between containers and platform virtualization

   - Concepts and terminology related to namespaces and cgroups

   - An example use of process resource isolation and management with network namespaces and cgroups
