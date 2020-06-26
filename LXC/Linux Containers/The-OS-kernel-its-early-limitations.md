---
layout: default
title: Introduction to Linux Containers
parent: LXC Hands-On Workshop
nav_order: 2
---

# The OS kernel and its early limitations

- The current state of Linux containers is a direct result of the problems that early OS designers were trying to solve – managing memory, I/O, and process scheduling in the most efficient way.

- In the past, only a single process could be scheduled for work, wasting precious CPU cycles if blocked on an I/O operation. The solution to this problem was to develop better CPU schedulers, so more work can be allocated in a fair way for maximum CPU utilization. Even though the modern schedulers, such as the Completely Fair Scheduler (CFS) in Linux do a great job of allocating fair amounts of time to each process, there's still a strong case for being able to give higher or lower priority to a process and its subprocesses. Traditionally, this can be accomplished by the nice() system call, or real-time scheduling policies, however, there are limitations to the level of granularity or control that can be achieved.

- Similarly, before the advent of virtual memory, multiple processes would allocate memory from a shared pool of physical memory. The virtual memory provided some form of memory isolation per process, in the sense that processes would have their own address space, and extend the available memory by means of a swap, but still there wasn't a good way of limiting how much memory each process and its children can use.

- To further complicate the matter, running different workloads on the same physical server usually resulted in a negative impact on all running services. A memory leak or a kernel panic could cause one application to bring the entire operating system down. For example, a web server that is mostly memory bound and a database service that is I/O heavy running together became problematic. In an effort to avoid such scenarios, system administrators would separate the various applications between a pool of servers, leaving some machines underutilized, especially at certain times during the day, when there was not much work to be done. This is a similar problem as a single running process blocked on I/O operation is a waste of CPU and memory resources.

- The solution to these problems is the use of hypervisor based virtualization, containers, or the combination of both.
