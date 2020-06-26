---
layout: default
title: User namespaces
parent: LXC Hands-On Workshop
nav_order: 8
---

# User namespaces

- The user namespaces allow a process inside a namespace to have a different user and group ID than that in the default namespace. 
- In the context of LXC, this allows for a process to run as root inside the container, while having a non-privileged ID outside. 
This adds a thin layer of security, because braking out for the container will result in a non-privileged user. This is possible because of kernel 3.8, 
which introduced the ability for non-privileged processes to create user namespaces.

- To create a new user namespace as a non-privileged user and have `root` inside, we can use the unshare utility. Let's install the latest version from source:

```
root@ubuntu:~# cd /usr/src/
root@ubuntu:/usr/src# wget https://www.kernel.org/pub/linux/utils/util-linux/v2.28/util-linux-2.28.tar.gz
root@ubuntu:/usr/src# tar zxfv util-linux-2.28.tar.gz
root@ubuntu:/usr/src# cd util-linux-2.28/
root@ubuntu:/usr/src/util-linux-2.28# ./configure
root@ubuntu:/usr/src/util-linux-2.28# make && make install
root@ubuntu:/usr/src/util-linux-2.28# unshare --map-root-user --user sh -c whoami
root
root@ubuntu:/usr/src/util-linux-2.28#

```
We can also use the `clone()` system call with the CLONE_NEWUSER flag to create a process in a user namespace, as demonstrated by the following program:

```
#define _GNU_SOURCE 
#include <stdlib.h> 
#include <stdio.h> 
#include <signal.h> 
#include <sched.h> 
 
static int childFunc(void *arg) 
{ 
    printf("UID inside the namespace is %ld\n", (long) 
    geteuid()); 
    printf("GID inside the namespace is %ld\n", (long) 
    getegid()); 
} 
 
static char child_stack[1024*1024]; 
 
int main(int argc, char *argv[]) 
{ 
    pid_t child_pid; 
 
    child_pid = clone(childFunc, child_stack +  
    (1024*1024),        
    CLONE_NEWUSER | SIGCHLD, NULL); 
 
    printf("UID outside the namespace is %ld\n", (long)       
    geteuid()); 
    printf("GID outside the namespace is %ld\n", (long)      
    getegid()); 
    waitpid(child_pid, NULL, 0); 
    exit(EXIT_SUCCESS); 
} 
```

After compilation and execution, the output looks similar to this when run as root - UID of 0:

```
root@server:~# gcc user_namespace.c -o user_namespace
root@server:~# ./user_namespace
UID outside the namespace is 0
GID outside the namespace is 0
UID inside the namespace is 65534
GID inside the namespace is 65534
root@server:~#

```


