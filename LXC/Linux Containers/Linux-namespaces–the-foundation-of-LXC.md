---
layout: default
title: Linux namespaces – the foundation of LXC
parent: LXC Hands-On Workshop
nav_order: 4
---


# Linux namespaces – the foundation of LXC

- Namespaces are the foundation of lightweight process virtualization. They enable a process and its children to have different views of the underlying system. 
This is achieved by the addition of the `unshare(`) and `setns()` system calls,

- and the inclusion of six new constant flags passed to the `clone()`, `unshare()`, and `setns()` system calls:
     
   - `clone()`: This creates a new process and attaches it to a new specified namespace

   -  `unshare()`: This attaches the current process to a new specified namespace

   - `setns()`: This attaches a process to an already existing namespace
   
- There are six namespaces currently in use by LXC, with more being developed:

    - Mount namespaces, specified by the `CLONE_NEWNS` flag

    - UTS namespaces, specified by the `CLONE_NEWUTS` flag

    - IPC namespaces, specified by the `CLONE_NEWIPC` flag

    - PID namespaces, specified by the `CLONE_NEWPID` flag

    - User namespaces, specified by the `CLONE_NEWUSER` flag

    - Network namespaces, specified by the `CLONE_NEWNET` flag
    
- Let's have a look at each in more detail and see some userspace examples, to help us better understand what happens under the hood.
