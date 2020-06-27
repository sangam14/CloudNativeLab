---
layout: default
title:  The cgroup freezer subsystem
parent: LXC Hands-On Workshop
nav_order: 14
---


# The cgroup freezer subsystem

The freezer subsystem can be used to suspend the current state of running tasks for the purposes of analyzing them, or to create a checkpoint that can be used to migrate the process to a different server. Another use case is when a process is negatively impacting the system and needs to be temporarily paused, without losing its current state data.

The next example shows how to suspend the execution of the top process, check its state, and then resume it.

First, mount the freezer subsystem and create the new hierarchy:

```
root@server:~# mkdir /cgroup/freezer
root@server:~# mount -t cgroup -o freezer freezer /cgroup/freezer
root@server:~# mkdir /cgroup/freezer/frozen_group
root@server:~# cat /proc/mounts | grep freezer
freezer /cgroup/freezer cgroup rw,relatime,freezer,release_agent=/run/cgmanager/agents/cgm-release-agent.freezer 0 0
root@server:~#

```

In a new terminal, start the top process and observe how it periodically refreshes. Back in the original terminal, add the PID of top to the frozen_group task file and observe its state:

```
root@server:~# echo 25731 > /cgroup/freezer/frozen_group/tasks
root@server:~# cat /cgroup/freezer/frozen_group/freezer.state
THAWED
root@server:~#


```
To freeze the process, echo the following:

```
root@server:~# echo FROZEN > /cgroup/freezer/frozen_group/freezer.state
root@server:~# cat /cgroup/freezer/frozen_group/freezer.state
FROZEN
root@server:~# cat /proc/25s731/status | grep -i state
State:      D (disk sleep)
root@server:~#

```

Notice how the top process output is not refreshing anymore, and upon inspection of its status file, you can see that it is now in the blocked state.

To resume it, execute the following:

```
root@server:~# echo THAWED > /cgroup/freezer/frozen_group/freezer.state
root@server:~# cat /proc/29328/status  | grep -i state
State:  S (sleeping)
root@server:~#


```

Inspecting the frozen_group hierarchy yields the following files:

```
root@server:~# ls -la /cgroup/freezer/frozen_group/
drwxr-xr-x 2 root root 0 Aug 25 20:50 .
drwxr-xr-x 4 root root 0 Aug 19 21:02 ..
-rw-r--r-- 1 root root 0 Aug 25 20:50 cgroup.clone_children
--w--w--w- 1 root root 0 Aug 25 20:50 cgroup.event_control
-rw-r--r-- 1 root root 0 Aug 25 20:50 cgroup.procs
-r--r--r-- 1 root root 0 Aug 25 20:50 freezer.parent_freezing
-r--r--r-- 1 root root 0 Aug 25 20:50 freezer.self_freezing
-rw-r--r-- 1 root root 0 Aug 25 21:00 freezer.state
-rw-r--r-- 1 root root 0 Aug 25 20:50 notify_on_release
-rw-r--r-- 1 root root 0 Aug 25 20:59 tasks
root@server:~#

```
The few files of interest are described in the following table:

| File         |  Description    | 
|:-------------|:------------------|
| ` freezer.parent_freezing`  | Shows the parent-state. Shows 0 if none of the cgroup's ancestors is FROZEN; otherwise 1. | 
| `freezer.self_freezing` |  Shows the self-state. Shows 0 if the self-state is THAWED; otherwise, 1. | 
| ` freezer.state `  |  Sets the self-state of the cgroup to either THAWED or FROZEN. | 
| `tasks`  | Attaches tasks to the cgroup. | 

