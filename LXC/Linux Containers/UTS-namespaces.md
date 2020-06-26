---
layout: default
title: UTS namespaces
parent: LXC Hands-On Workshop
nav_order: 6
---


# UTS namespaces

- Unix Timesharing (UTS) namespaces provide isolation for the hostname and domain name, so that each LXC container can maintain its own identifier as returned by the hostname `-f` command.
This is needed for most applications that rely on a properly set hostname.

- To create a bash session in a new UTS namespace, we can use the unshare utility again, which uses the `unshare()` system call to create the namespace and the execve() system call to 
execute bash in it:

```
root@server:~# hostname
server
root@server:~# unshare -u /bin/bash
root@server:~# hostname uts-namespace
root@server:~# hostname
uts-namespace
root@server:~# cat /proc/sys/kernel/hostname
uts-namespace
root@server:~#

```

As the preceding output shows, the hostname inside the namespace is now `uts-namespace`.

# Next, from a different terminal, check the hostname again to make sure it has not changed:
```

root@server:~# hostname
server
root@server:~#

```
As expected, the hostname only changed in the new UTS namespace.

# To see the actual system calls that the unshare command uses, we can run the strace utility:

```
root@server:~# strace -s 2000 -f unshare -u /bin/bash
...
unshare(CLONE_NEWUTS)                   = 0
getgid()                                = 0
setgid(0)                               = 0
getuid()                                = 0
setuid(0)                               = 0
execve("/bin/bash", ["/bin/bash"], [/* 15 vars */]) = 0
...

```
From the output we can see that the unshare command is indeed using the `unshare()` and `execve()` system calls 
and the `CLONE_NEWUTS` flag to specify new UTS namespace.


# IPC namespaces

- The Interprocess Communication (IPC) namespaces provide isolation for a set of IPC and synchronization facilities. 
- These facilities provide a way of exchanging data and synchronizing the actions between threads and processes. 
- They provide primitives such as semaphores, file locks, and mutexes among others, that are needed to have true process separation in a container.
