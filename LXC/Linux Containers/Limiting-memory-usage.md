---
layout: default
title: Limiting memory usage
parent: LXC Hands-On Workshop
nav_order: 12
---


# Limiting memory usage

- The memory subsystem controls how much memory is presented to and available for use by processes. This can be particularly useful in multitenant environments where better control over how much memory a user process can utilize is needed, or to limit memory hungry applications. Containerized solutions like LXC can use the memory subsystem to manage the size of the instances, without needing to restart the entire container.

- The memory subsystem performs resource accounting, such as tracking the utilization of anonymous pages, file caches, swap caches, 
and general hierarchical accounting, all of which presents an overhead. Because of this, the memory cgroup is disabled by default on some Linux distributions. 
If the following commands below fail you'll need to enable it, by specifying the following GRUB parameter and restarting:

```

root@server:~# vim /etc/default/grub
RUB_CMDLINE_LINUX_DEFAULT="cgroup_enable=memory"
root@server:~# grub-update && reboot


```
First, let's mount the memory cgroup:

```
root@server:~# mkdir -p /cgroup/memory
root@server:~# mount -t cgroup -o memory memory /cgroup/memory
root@server:~# cat /proc/mounts | grep memory
memory /cgroup/memory cgroup rw, relatime, memory, release_agent=/run/cgmanager/agents/cgm-release-agent.memory 0 0
root@server:~#

```
Then set the app1 memory to 1 GB:

```

root@server:~# mkdir /cgroup/memory/app1
root@server:~# echo 1G > /cgroup/memory/app1/memory.limit_in_bytes
root@server:~# cat /cgroup/memory/app1/memory.limit_in_bytes
1073741824
root@server:~# pidof app1 | while read PID; do echo $PID >> /cgroup/memory/app1/tasks; done
root@server:~#


```

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/mount-app1.jpg)

The memory hierarchy for the app1 process

Similar to the blkio subsystem, the tasks file is used to specify the PID of the processes we are adding to the cgroup hierarchy, and the memory.limit_in_bytes specifies how much memory is to be made available in bytes.

The app1 memory hierarchy contains the following files:

```
root@server:~# ls -la /cgroup/memory/app1/
drwxr-xr-x 2 root root 0 Aug 24 22:05 .
drwxr-xr-x 3 root root 0 Aug 19 21:02 ..
-rw-r--r-- 1 root root 0 Aug 24 22:05 cgroup.clone_children
--w--w--w- 1 root root 0 Aug 24 22:05 cgroup.event_control
-rw-r--r-- 1 root root 0 Aug 24 22:05 cgroup.procs
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.failcnt
--w------- 1 root root 0 Aug 24 22:05 memory.force_empty
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.kmem.failcnt
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.kmem.limit_in_bytes
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.kmem.max_usage_in_bytes
-r--r--r-- 1 root root 0 Aug 24 22:05 memory.kmem.slabinfo
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.kmem.tcp.failcnt
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.kmem.tcp.limit_in_bytes
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.kmem.tcp.max_usage_in_bytes
-r--r--r-- 1 root root 0 Aug 24 22:05 memory.kmem.tcp.usage_in_bytes
-r--r--r-- 1 root root 0 Aug 24 22:05 memory.kmem.usage_in_bytes
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.limit_in_bytes
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.max_usage_in_bytes
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.move_charge_at_immigrate
-r--r--r-- 1 root root 0 Aug 24 22:05 memory.numa_stat
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.oom_control
---------- 1 root root 0 Aug 24 22:05 memory.pressure_level
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.soft_limit_in_bytes
-r--r--r-- 1 root root 0 Aug 24 22:05 memory.stat
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.swappiness
-r--r--r-- 1 root root 0 Aug 24 22:05 memory.usage_in_bytes
-rw-r--r-- 1 root root 0 Aug 24 22:05 memory.use_hierarchy
-rw-r--r-- 1 root root 0 Aug 24 22:05 tasks
root@server:~#


```

The files and their function in the memory subsystem are described in the following table:

| File         | Description    | 
|:-------------|:------------------|
| `memory.failcnt `  | Shows the total number of memory limit hits  | 
| `memory.force_empty` | If set to 0, frees memory used by tasks  | 
| ` memory.kmem.failcnt `  |   Shows the total number of kernel memory limit hits | 
| `memory.kmem.limit_in_bytes `  | Sets or shows kernel memory hard limit | 
| `memory.kmem.max_usage_in_bytes `| Shows maximum kernel memory usage | 
| `memory.kmem.tcp.failcnt ` | Shows the number of TCP buffer memory limit hits | 
| `memory.kmem.tcp.limit_in_bytes` | Sets or shows hard limit for TCP buffer memory| 
| `memory.kmem.tcp.max_usage_in_bytes `  |Shows maximum TCP buffer memory usage | 
| `memory.kmem.tcp.usage_in_bytes `  | Shows current TCP buffer memory |
| `memory.kmem.usage_in_bytes` | Shows current kernel memory | 
| ` memory.limit_in_bytes ` | Sets or shows memory usage limit | 
| ` memory.max_usage_in_bytes ` | Shows maximum memory usage | 
| `memory.move_charge_at_immigrate ` | Sets or shows controls of moving charges| 
| ` memory.numa_stat` |  Shows the number of memory usage per NUMA node | 
| ` memory.oom_control` | Sets or shows the OOM controls | 
| `memory.pressure_level ` | Sets memory pressure notifications | 
| ` memory.soft_limit_in_bytes` | Sets or shows soft limit of memory usage | 
| ` memory.stat` | Shows various statistics | 
| `memory.swappiness ` | Sets or shows swappiness level | 
| ` memory.usage_in_bytes` | Shows current memory usage | 
| ` memory.use_hierarchy` | Sets memory reclamation from child processes | 
| ` task` | Attaches tasks to the cgroup | 

Limiting the memory available to a process might trigger the Out of Memory (OOM) killer, which might kill the running task. 
If this is not the desired behavior and you prefer the process to be suspended waiting for memory to be freed, the OOM killer can be disabled:

```
root@server:~# cat /cgroup/memory/app1/memory.oom_control
oom_kill_disable 0
under_oom 0
root@server:~# echo 1 > /cgroup/memory/app1/memory.oom_control
root@server:~#

```

The memory cgroup presents a wide slew of accounting statistics in the memory.stat file, which can be of interest:

```

root@server:~# head /cgroup/memory/app1/memory.stat
cache 43325     # Number of bytes of page cache memory
rss 55d43       # Number of bytes of anonymous and swap cache memory
rss_huge 0      # Number of anonymous transparent hugepages
mapped_file 2   # Number of bytes of mapped file
writeback 0     # Number of bytes of cache queued for syncing
pgpgin 0        # Number of charging events to the memory cgroup
pgpgout 0       # Number of uncharging events to the memory cgroup
pgfault 0       # Total number of page faults
pgmajfault 0    # Number of major page faults
inactive_anon 0 # Anonymous and swap cache memory on inactive LRU list


```
If you need to start a new task in the app1 memory hierarchy you can move the current shell process into the tasks file, 
and all other processes started in this shell will be direct descendants and inherit the same cgroup properties:

```
root@server:~# echo $$ > /cgroup/memory/app1/tasks
root@server:~# echo "The memory limit is now applied to all processes started from this shell"

```
