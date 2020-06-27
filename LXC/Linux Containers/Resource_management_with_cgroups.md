---
layout: default
title: Resource management with cgroups
parent: LXC Hands-On Workshop
nav_order: 10
---


# Resource management with cgroups

- Cgroups are kernel features that allows fine-grained control over resource allocation for a single process, or a group of processes, called tasks. In the context of LXC this is quite important, because it makes it possible to assign limits to how much memory, CPU time, or I/O, any given container can use.

The cgroups we are most interested in are described in the following table:

| Subsystem     | Description     |  Defined in    | 
|:-------------|:------------------|:------------------|
| `cpu`  |   Allocates CPU time for tasks | `kernel/sched/core.c` |
| `cpuacct` |  Accounts for CPU usage  | `kernel/sched/core.c`|
| `cpuset `  |  Assigns CPU cores to tasks  | `kernel/cpuset.c`|
| `memory`  | Allocates memory for tasks | `mm/memcontrol.c`|
| `blkio `| Limits the I/O access to devices | `block/blk-cgroup.c`|
| `devices` | Allows/denies access to devices | `security/device_cgroup.c`|
| `freezer` | Suspends/resumes tasks | `kernel/cgroup_freezer.c`|
| `net_cls`  | Tags network packets | `net/sched/cls_cgroup.c`|
| `net_prio`  | Prioritizes network traffic |` net/core/netprio_cgroup.c`|
| `hugetlb` | Limits the HugeTLB | `mm/hugetlb_cgroup.c` |

- Cgroups are organized in hierarchies, represented as directories in a Virtual File System (VFS). Similar to process hierarchies, where every process is a descendent of the init or systemd process, cgroups inherit some of the properties of their parents. Multiple cgroups hierarchies can exist on the system, each one representing a single or group of resources. It is possible to have hierarchies that combine two or more subsystems, for example, memory and I/O, and tasks assigned to a group will have limits applied on those resources.

Note
```
If you are interested in how the different subsystems are implemented in the kernel, install the kernel source and have a look at the C files,
shown in the third column of the table.
```
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/cgroup1.jpg)

Cgroups can be used in two ways:

   - By manually manipulating files and directories on a mounted VFS

   - Using userspace tools provided by various packages such as cgroup-bin on Debian/Ubuntu and libcgroup on RHEL/CentOS

Let's have a look at few practical examples on how to use cgroups to limit resources. This will help us get a better understanding of how containers work.
