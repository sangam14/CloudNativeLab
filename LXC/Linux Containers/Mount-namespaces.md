---
layout: default
title: Mount namespaces
parent: LXC Hands-On Workshop
nav_order: 5
---


# Mount namespaces

Mount namespaces first appeared in kernel 2.4.19 in 2002 and provided a separate view of the filesystem mount points for the process and its children. 
When mounting or unmounting a filesystem, the change will be noticed by all processes because they all share the same default namespace. When the `CLONE_NEWNS`
flag is passed to the clone() system call, the new process gets a copy of the calling process mount tree that it can then change without affecting 
the parent process. From that point on, all mounts and unmounts in the default namespace will be visible in the new namespace, but changes in the per-process 
mount namespaces will not be noticed outside of it.

The `clone()` prototype is as follows:

```
#define _GNU_SOURCE 
#include <sched.h> 
int clone(int (*fn)(void *), void *child_stack, int flags, void *arg); 

```

An example call that creates a child process in a new mount namespace looks like this:
```
child_pid = clone(childFunc, child_stack + STACK_SIZE, CLONE_NEWNS | SIGCHLD, argv[1]); 

```
- When the child process is created, it executes the childFunc function, which will perform its work in the new mount namespace.

- The util-linux package provides userspace tools that implement the unshare() call, which effectively unshares the indicated namespaces from the parent process.

To illustrate this:

# First open a terminal and create a directory in `/tmp` as follows:
```
root@server:~# mkdir /tmp/mount_ns
root@server:~#

```
# Next, move the current bash process to its own mount namespace by passing the `mount` flag to `unshare`:

```
root@server:~# unshare -m /bin/bash
root@server:~#

```
# The `bash` process is now in a separate namespace. Let's check the associated inode number of the namespace:

```
root@server:~# readlink /proc/$$/ns/mnt
mnt:[4026532211]
root@server:~#

```
# Next, create a temporary mount point:

```
root@server:~# mount -n -t tmpfs tmpfs /tmp/mount_ns
root@server:~#


```

# Also, make sure you can see the mount point from the newly created namespace:

```
root@server:~# df -h | grep mount_ns
tmpfs           3.9G     0  3.9G   0% /tmp/mount_ns
root@server:~# cat /proc/mounts | grep mount_ns
tmpfs /tmp/mount_ns tmpfs rw,relatime 0 0
root@server:~#

```
 As expected, the mount point is visible because it is part of the namespace we created and the current bash process is running from
 # Next, start a new terminal session and display the namespace inode ID from it:

```
root@server:~# readlink /proc/$$/ns/mnt
mnt:[4026531840]
root@server:~#

```
Notice how it's different from the mount namespace on the other terminal.
# Finally, check if the mount point is visible in the new terminal:
```
root@server:~# cat /proc/mounts | grep mount_ns
root@server:~# df -h | grep mount_ns
root@server:~#

```

Not surprisingly, the mount point is not visible from the default namespace.

# Note
```
In the context of LXC, mount namespaces are useful because they provide a way for a different filesystem layout to exist inside the container. 
It's worth mentioning that before the mount namespaces, a similar process confinement could be achieved with the chroot() system call,
however chroot does not provide the same per-process isolation as mount namespaces do
```

