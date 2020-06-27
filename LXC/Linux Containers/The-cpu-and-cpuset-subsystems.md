---
layout: default
title: The cpu and cpuset subsystems
parent: LXC Hands-On Workshop
nav_order: 13
---


# The cpu and cpuset subsystems

- The cpu subsystem schedules CPU time to cgroup hierarchies and their tasks. It provides finer control over CPU execution time than the default behavior of the CFS.

- The cpuset subsystem allows for assigning CPU cores to a set of tasks, similar to the taskset command in Linux.

- The main benefits that the cpu and cpuset subsystems provide are better utilization per processor core for highly CPU bound applications. They also allow for distributing load between cores that are otherwise idle at certain times of the day. In the context of multitenant environments, running many LXC containers, cpu and cpuset cgroups allow for creating different instance sizes and container flavors, for example exposing only a single core per container, with 40 percent scheduled work time.

As an example, let's assume we have two processes app1 and app2, and we would like app1 to use 60 percent of the CPU time and app2 only 40 percent. 
We start by mounting the cgroup VFS:

```
root@server:~# mkdir -p /cgroup/cpu
root@server:~# mount -t cgroup -o cpu cpu /cgroup/cpu
root@server:~# cat /proc/mounts | grep cpu
cpu /cgroup/cpu cgroup rw, relatime, cpu, release_agent=/run/cgmanager/agents/cgm-release-agent.cpu 0 0
```

Then we create two child hierarchies:
```
root@server:~# mkdir /cgroup/cpu/limit_60_percent
root@server:~# mkdir /cgroup/cpu/limit_40_percent

```

Also assign CPU shares for each, where app1 will get 60 percent and app2 will get 40 percent of the scheduled time:

```
root@server:~# echo 600 > /cgroup/cpu/limit_60_percent/cpu.shares
root@server:~# echo 400 > /cgroup/cpu/limit_40_percent/cpu.shares

```

Finally, we move the PIDs in the tasks files:

```
root@server:~# pidof app1 | while read PID; do echo $PID >> /cgroup/cpu/limit_60_percent/tasks; done
root@server:~# pidof app2 | while read PID; do echo $PID >> /cgroup/cpu/limit_40_percent/tasks; done
root@server:~#

```

The cpu subsystem contains the following control files:

```

root@server:~# ls -la /cgroup/cpu/limit_60_percent/
drwxr-xr-x 2 root root 0 Aug 25 15:13 .
drwxr-xr-x 4 root root 0 Aug 19 21:02 ..
-rw-r--r-- 1 root root 0 Aug 25 15:13 cgroup.clone_children
--w--w--w- 1 root root 0 Aug 25 15:13 cgroup.event_control
-rw-r--r-- 1 root root 0 Aug 25 15:13 cgroup.procs
-rw-r--r-- 1 root root 0 Aug 25 15:13 cpu.cfs_period_us
-rw-r--r-- 1 root root 0 Aug 25 15:13 cpu.cfs_quota_us
-rw-r--r-- 1 root root 0 Aug 25 15:14 cpu.shares
-r--r--r-- 1 root root 0 Aug 25 15:13 cpu.stat
-rw-r--r-- 1 root root 0 Aug 25 15:13 notify_on_release
-rw-r--r-- 1 root root 0 Aug 25 15:13 tasks
root@server:~#


```
Here's a brief explanation of each:

| File   | Description   | 
|:-------------|:------------------|
| ` cpu.cfs_period_us `  |  CPU resource reallocation in microseconds | 
| ` cpu.cfs_quota_us ` |   Run duration of tasks in microseconds during one `cpu.cfs_perious_us` period  | 
| ` cpu.shares `  |   Relative share of CPU time available to the tasks | 
| `cpu.stat `  | Shows CPU time statistics | 
| ` tasks `| Attaches tasks to the cgroup  | 


The `cpu.stat` file is of particular interest:


```

root@server:~# cat /cgroup/cpu/limit_60_percent/cpu.stat
nr_periods 0        # number of elapsed period intervals, as specified in
                # cpu.cfs_period_us
nr_throttled 0      # number of times a task was not scheduled to run
                # because of quota limit
throttled_time 0    # total time in nanoseconds for which tasks have been
                # throttled
root@server:~#

```

To demonstrate how the cpuset subsystem works, let's create cpuset hierarchies named app1, containing CPUs 0 and 1. The app2 cgroup will contain only CPU 1:

```
root@server:~# mkdir /cgroup/cpuset
root@server:~# mount -t cgroup -o cpuset cpuset /cgroup/cpuset
root@server:~# mkdir /cgroup/cpuset/app{1..2}
root@server:~# echo 0-1 > /cgroup/cpuset/app1/cpuset.cpus
root@server:~# echo 1 > /cgroup/cpuset/app2/cpuset.cpus
root@server:~# pidof app1 | while read PID; do echo $PID >> /cgroup/cpuset/app1/tasks limit_60_percent/tasks; done
root@server:~# pidof app2 | while read PID; do echo $PID >> /cgroup/cpuset/app2/tasks limit_40_percent/tasks; done
root@server:~#



```

To check if the app1 process is pinned to CPU 0 and 1, we can use:

```
root@server:~# taskset -c -p $(pidof app1)
pid 8052's current affinity list: 0,1
root@server:~# taskset -c -p $(pidof app2)
pid 8052's current affinity list: 1
root@server:~#

```
The cpuset app1 hierarchy contains the following files:

```

root@server:~# ls -la /cgroup/cpuset/app1/
drwxr-xr-x 2 root root 0 Aug 25 16:47 .
drwxr-xr-x 5 root root 0 Aug 19 21:02 ..
-rw-r--r-- 1 root root 0 Aug 25 16:47 cgroup.clone_children
--w--w--w- 1 root root 0 Aug 25 16:47 cgroup.event_control
-rw-r--r-- 1 root root 0 Aug 25 16:47 cgroup.procs
-rw-r--r-- 1 root root 0 Aug 25 16:47 cpuset.cpu_exclusive
-rw-r--r-- 1 root root 0 Aug 25 17:57 cpuset.cpus
-rw-r--r-- 1 root root 0 Aug 25 16:47 cpuset.mem_exclusive
-rw-r--r-- 1 root root 0 Aug 25 16:47 cpuset.mem_hardwall
-rw-r--r-- 1 root root 0 Aug 25 16:47 cpuset.memory_migrate
-r--r--r-- 1 root root 0 Aug 25 16:47 cpuset.memory_pressure
-rw-r--r-- 1 root root 0 Aug 25 16:47 cpuset.memory_spread_page
-rw-r--r-- 1 root root 0 Aug 25 16:47 cpuset.memory_spread_slab
-rw-r--r-- 1 root root 0 Aug 25 16:47 cpuset.mems
-rw-r--r-- 1 root root 0 Aug 25 16:47 cpuset.sched_load_balance
-rw-r--r-- 1 root root 0 Aug 25 16:47 cpuset.sched_relax_domain_level
-rw-r--r-- 1 root root 0 Aug 25 16:47 notify_on_release
-rw-r--r-- 1 root root 0 Aug 25 17:13 tasks
root@server:~#


```

A brief description of the control files is as follows:

| File   | Description   | 
|:-------------|:------------------|
| `  cpuset.cpu_exclusive  ` |   Checks if other cpuset hierarchies share the settings defined in the current group  | 
| ` cpuset.cpus  `  |   List of the physical numbers of the CPUs on which processes in that cpuset are allowed to execute  | 
| ` cpuset.mem_exclusive  `  |  Should the cpuset have exclusive use of its memory nodes   | 
| `  cpuset.mem_hardwall `  |    Checks if each tasks' user allocation be kept separate | 
| ` cpuset.memory_migrate  `  |  Checks if a page in memory should migrate to a new node if the values in cpuset.mems change   | 
| `  cpuset.memory_pressure  ` |   Contains running average of the memory pressure created by the processes  | 
| `  cpuset.memory_spread_page `  |   Checks if filesystem buffers should spread evenly across the memory nodes  | 
| `  cpuset.memory_spread_slab `  |   Checks if kernel slab caches for file I/O operations should spread evenly across the cpuset  | 
| ` cpuset.mems  `  |  Specifies the memory nodes that tasks in this cgroup are permitted to access   | 
| ` cpuset.sched_load_balance  `  |  Checks if the kernel balance should load across the CPUs in the cpuset by moving processes from overloaded CPUs to less utilized CPUs   | 
| `  cpuset.sched_relax_domain_level  ` |  Contains the width of the range of CPUs across which the kernel should attempt to balance loads   | 
| ` notify_on_release  `  |    Checks if the hierarchy should receive special handling after it is released and no process are using it | 
| `  tasks `  |   Attaches tasks to the cgroup  | 


