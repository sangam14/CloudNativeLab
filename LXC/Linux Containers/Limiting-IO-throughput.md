---
layout: default
title:  Limiting I/O throughput
parent: LXC Hands-On Workshop
nav_order: 11
---

# Limiting I/O throughput


Let's assume we have two applications running on a server that are heavily I/O bound: 
 - app1 and app2. We would like to give more bandwidth to app1 during the day and to app2 during the night. 
 - This type of I/O throughput prioritization can be accomplished using the blkio subsystem.

First, let's attach the blkio subsystem by mounting the cgroup VFS:
```

root@server:~# mkdir -p /cgroup/blkio
root@server:~# mount -t cgroup -o blkio blkio /cgroup/blkio
root@server:~# cat /proc/mounts | grep cgroup
blkio /cgroup/blkio cgroup rw, relatime, blkio, crelease_agent=/run/cgmanager/agents/cgm-release-agent.blkio 0 0
root@server:~#


```
Next, create two priority groups, which will be part of the same blkio hierarchy:
```
root@server:~# mkdir /cgroup/blkio/high_io
root@server:~# mkdir /cgroup/blkio/low_io
root@server:~#

```
We need to acquire the PIDs of the app1 and app2 processes and assign them to the high_io and low_io groups:
```
root@server:~# pidof app1 | while read PID; do echo $PID >> /cgroup/blkio/high_io/tasks; done 
root@server:~# pidof app2 | while read PID; do echo $PID >> /cgroup/blkio/low_io/tasks; done
root@server:~#

```
![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/blkio1.jpg)

The blkio hierarchy we've created

The tasks file is where we define what processes/tasks the limit should be applied on.

Finally, let's set a ratio of 10:1 for the high_io and low_io cgroups. Tasks in those cgroups will immediately use only the resources made available to them:

```
root@server:~# echo 1000 > /cgroup/blkio/high_io/blkio.weight
root@server:~# echo 100 > /cgroup/blkio/low_io/blkio.weight
root@server:~#

```

The `blkio.weight` file defines the weight of I/O access available to a process or group of processes, with values ranging from 100 to 1,000. In this example, the values of 1000 and 100 create a ratio of 10:1.

With this, the low priority application, app2 will use only about 10 percent of the I/O operations available, whereas the high priority application, app1, will use about 90 percent.

If you list the contents of the `high_io` directory on Ubuntu you will see the following files:

```
root@server:~# ls -la /cgroup/blkio/high_io/
drwxr-xr-x 2 root root 0 Jun 24 16:14 .
drwxr-xr-x 4 root root 0 Jun 19 21:14 ..
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_merged
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_merged_recursive
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_queued
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_queued_recursive
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_service_bytes
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_service_bytes_recursive
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_serviced
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_serviced_recursive
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_service_time
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_service_time_recursive
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_wait_time
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.io_wait_time_recursive
-rw-r--r-- 1 root root 0 Jun 24 16:14 blkio.leaf_weight
-rw-r--r-- 1 root root 0 Jun 24 16:14 blkio.leaf_weight_device
--w------- 1 root root 0 Jun 24 16:14 blkio.reset_stats
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.sectors
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.sectors_recursive
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.throttle.io_service_bytes
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.throttle.io_serviced
-rw-r--r-- 1 root root 0 Jun 24 16:14 blkio.throttle.read_bps_device
-rw-r--r-- 1 root root 0 Jun 24 16:14 blkio.throttle.read_iops_device
-rw-r--r-- 1 root root 0 Jun 24 16:14 blkio.throttle.write_bps_device
-rw-r--r-- 1 root root 0 Jun 24 16:14 blkio.throttle.write_iops_device
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.time
-r--r--r-- 1 root root 0 Jun 24 16:14 blkio.time_recursive
-rw-r--r-- 1 root root 0 Jun 24 16:49 blkio.weight
-rw-r--r-- 1 root root 0 Jun 24 17:01 blkio.weight_device
-rw-r--r-- 1 root root 0 Jun 24 16:14 cgroup.clone_children
--w--w--w- 1 root root 0 Jun 24 16:14 cgroup.event_control
-rw-r--r-- 1 root root 0 Jun 24 16:14 cgroup.procs
-rw-r--r-- 1 root root 0 Jun 24 16:14 notify_on_release
-rw-r--r-- 1 root root 0 Jun 24 16:14 tasks
root@server:~#

```

From the preceding output you can see that only some files are writeable. This depends on various OS settings, such as what I/O scheduler is being used.

We've already seen what the tasks and blkio.weight files are used for. The following is a short description of the most commonly used files in the blkio subsystem:



| Subsystem     | Description     | 
|:-------------|:------------------|
| `blkio.io_merged`  |  Total number of reads/writes, sync, or async merged into requests | 
| `blkio.io_queued` |  Total number of read/write, sync, or async requests queued up at any given time | 
| `blkio.io_service_bytes `  |  The number of bytes transferred to or from the specified device  | 
| `blkio.io_serviced`  | The number of I/O operations issued to the specified device | 
| `blkio.io_service_time `| Total amount of time between request dispatch and request completion in nanoseconds for the specified device | 
| `blkio.io_wait_time` | Total amount of time the I/O operations spent waiting in the scheduler queues for the specified device | 
| `blkio.leaf_weight` | Similar to `blkio.weight` and can be applied to the Completely Fair Queuing (CFQ) I/O scheduler| 
| `blkio.reset_stats`  | Writing an integer to this file will reset all statistics | 
| `blkio.sectors`  | The number of sectors transferred to or from the specified device |
| `blkio.throttle.io_service_bytes` | The number of bytes transferred to or from the disk | 
| `blkio.throttle.io_serviced` | The number of I/O operations issued to the specified disk | 
| `blkio.time` | The disk time allocated to a device in milliseconds | 
| `blkio.weight` | Specifies weight for a cgroup hierarchy | 
| `blkio.weight_device` | Same as `blkio.weight`, but specifies a block device to apply the limit on | 
| `tasks` |  Attach tasks to the cgroup | 

# Tip

One thing to keep in mind is that writing to the files directly to make changes will not persist after the server restarts.
Later in this LXC Workshop series , you will learn how to use the userspace tools to generate persistent configuration files.
