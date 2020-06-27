---
layout: default
title:  Managing resources with systemd
parent: LXC Hands-On Workshop
nav_order: 16
---


# Managing resources with systemd

- With the increased adoption of systemd as an init system, new ways of manipulating cgroups were introduced. For example, if the cpu controller is enabled in the kernel, systemd will create a cgroup for each service by default. This behavior can be changed by adding or removing cgroup subsystems in the configuration file of systemd, usually found at /etc/systemd/system.conf.

- If multiple services are running on the server, the CPU resources will be shared equally among them by default, because systemd assigns equal weights to each. To change this behavior for an application, we can edit its service file and define the CPU shares, allocated memory, and I/O.

The following example demonstrates how to change the CPU shares, memory, and I/O limits for the nginx process:

```
root@server:~# vim /etc/systemd/system/nginx.service
.include /usr/lib/systemd/system/httpd.service
[Service]
CPUShares=2000
MemoryLimit=1G
BlockIOWeight=100


```
To apply the changes first reload systemd, then nginx:
```
root@server:~#  systemctl daemon-reload
root@server:~#  systemctl restart httpd.service
root@server:~#  
```
This will create and update the necessary control files in /sys/fs/cgroup/systemd and apply the limits.
