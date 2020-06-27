---
layout: default
title:  Installing and Running LXC on Linux Systems
parent: LXC Hands-On Workshop
nav_order: 17
---

# Installing and Running LXC on Linux Systems

LXC takes advantage of the kernel namespaces and cgroups to create process isolation we call containers, as we saw in the previous chapter. As such, LXC is not a separate software component in the Linux kernel, but rather a set of userspace tools, the liblxc library, and various language bindings.

In this workshop series , we'll cover the following topics:

   - Installing LXC on Ubuntu and CentOS using distribution packages

   - Compiling and installing LXC from source code

   - Building and starting containers using the provided templates and configuration files

   - Manually building the root filesystem and configuration files using tools such as `debootstrap` and `yum `

