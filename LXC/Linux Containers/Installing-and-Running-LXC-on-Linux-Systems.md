---
layout: default
title:  Installing and Running LXC on Linux Systems
parent: LXC Hands-On Workshop
nav_order: 17
---

# Installing and Running LXC on Linux Systems

LXC takes advantage of the kernel namespaces and cgroups to create process isolation we call containers, as we saw in the previous section . As such, LXC is not a separate software component in the Linux kernel, but rather a set of userspace tools, the liblxc library, and various language bindings.

In this workshop series , we'll cover the following topics:

   - Installing LXC on Ubuntu and CentOS using distribution packages

   - Compiling and installing LXC from source code

   - Building and starting containers using the provided templates and configuration files

   - Manually building the root filesystem and configuration files using tools such as `debootstrap` and `yum `
   
   
# Installing LXC on Ubuntu 

step 1 :- 
```
apt-get update -y
```
step 2 :- Install LXC
By default, LXC is available in the Ubuntu 16.04 default repository. You can install it by running the following command:
```
apt-get install lxc lxc-templates -y
```
After installing LXC, you can check the LXC using the following command:
```
lxc-checkconfig 
```
If everything fine, you should see the following output:
```
Kernel configuration not found at /proc/config.gz; searching...
Kernel configuration found at /boot/config-4.2.0-27-generic
--- Namespaces ---
Namespaces: enabled
Utsname namespace: enabled
Ipc namespace: enabled
Pid namespace: enabled
User namespace: enabled
Network namespace: enabled
Multiple /dev/pts instances: enabled

--- Control groups ---
Cgroup: enabled
Cgroup clone_children flag: enabled
Cgroup device: enabled
Cgroup sched: enabled
Cgroup cpu account: enabled
Cgroup memory controller: enabled
Cgroup cpuset: enabled

--- Misc ---
Veth pair device: enabled
Macvlan: enabled
Vlan: enabled
Bridges: enabled
Advanced netfilter: enabled
CONFIG_NF_NAT_IPV4: enabled
CONFIG_NF_NAT_IPV6: enabled
CONFIG_IP_NF_TARGET_MASQUERADE: enabled
CONFIG_IP6_NF_TARGET_MASQUERADE: enabled
CONFIG_NETFILTER_XT_TARGET_CHECKSUM: enabled
FUSE (for use with lxcfs): enabled

--- Checkpoint/Restore ---
checkpoint restore: enabled
CONFIG_FHANDLE: enabled
CONFIG_EVENTFD: enabled
CONFIG_EPOLL: enabled
CONFIG_UNIX_DIAG: enabled
CONFIG_INET_DIAG: enabled
CONFIG_PACKET_DIAG: enabled
CONFIG_NETLINK_DIAG: enabled
File capabilities: enabled

Note : Before booting a new kernel, you can check its configuration
usage : CONFIG=/path/to/config /usr/bin/lxc-checkconfig
```
# Create LXC Container

LXC comes with lots of ready-made templates for creating Linux containers. You can list out them by running the following command:

```
ls /usr/share/lxc/templates/

```
Output:
```
lxc-alpine    lxc-archlinux  lxc-centos  lxc-debian    lxc-fedora  lxc-openmandriva  lxc-oracle  lxc-slackware   lxc-sshd    lxc-ubuntu-cloud
lxc-altlinux  lxc-busybox    lxc-cirros  lxc-download  lxc-gentoo  lxc-opensuse      lxc-plamo   lxc-sparclinux  lxc-ubuntu

```
Now, create your first Ubuntu container with the following command:
```
lxc-create -n new-container -t ubuntu

```
Once the container created succesfully, you should see the following output:
```
Download complete
Copy /var/cache/lxc/xenial/rootfs-amd64 to /var/lib/lxc/new-container/rootfs ... 
Copying rootfs to /var/lib/lxc/new-container/rootfs ...
Generating locales (this might take a while)...
  en_IN.UTF-8... done
Generation complete.
Creating SSH2 RSA key; this may take some time ...
2048 SHA256:kiRG9aMIT87ZRJ/+RJoKh9dO7ykS0MNQmNKlbgDveiM root@mail.distcourt.local (RSA)
Creating SSH2 DSA key; this may take some time ...
1024 SHA256:Y0Ajo4RuUP3ty+BJOmHrQPscF+9oxDSfgLTALBE3/Yg root@mail.distcourt.local (DSA)
Creating SSH2 ECDSA key; this may take some time ...
256 SHA256:TsHzuCEgoRAMobpdUJCVCyRQo2YgKpGEZLZp4OF1iBw root@mail.distcourt.local (ECDSA)
Creating SSH2 ED25519 key; this may take some time ...
256 SHA256:wehDNjEy/AxVknDWj7BFTtvGKUnVXKLcmTTbm63OyM0 root@mail.distcourt.local (ED25519)
invoke-rc.d: could not determine current runlevel
invoke-rc.d: policy-rc.d denied execution of start.

Current default time zone: 'Etc/UTC'
Local time is now:      Mon Jul 30 07:40:47 UTC 2018.
Universal Time is now:  Mon Jul 30 07:40:47 UTC 2018.


##
# The default user is 'ubuntu' with password 'ubuntu'!
# Use the 'sudo' command to run tasks as root in the container.
##

```
You should see that the Ubuntu container created with the default user ubuntu and password ubuntu.

You can now list the created container with the following command:

```
lxc-ls 

```
Output:
```
new-container

```
# Install LXC Web Panel

LXC Web Panel is a GUI management tool to manage Linux containers. You can create, start, stop, clone, delete and restart Linux container from the web browser using LXC web panel.

You can install the LXC web panel to your system by running the following command:
```
wget https://lxc-webpanel.github.io/tools/install.sh -O - | bash

```

This command will download and install LXC web panel automatically to your system:
```
Installing requirement...
+ Installing Python pip
Extracting templates from packages: 100%
| + Flask Python...
Collecting flask==0.9
  Downloading https://files.pythonhosted.org/packages/49/0a/fe5021b35436202d3d4225a766f3bdc7fb51521ad89e73c5162db36cdbc7/Flask-0.9.tar.gz (481kB)
    100% |################################| 491kB 598kB/s 
Collecting Werkzeug>=0.7 (from flask==0.9)
  Downloading https://files.pythonhosted.org/packages/20/c4/12e3e56473e52375aa29c4764e70d1b8f3efa6682bef8d0aae04fe335243/Werkzeug-0.14.1-py2.py3-none-any.whl (322kB)
    100% |################################| 327kB 517kB/s 
Collecting Jinja2>=2.4 (from flask==0.9)
  Downloading https://files.pythonhosted.org/packages/7f/ff/ae64bacdfc95f27a016a7bed8e8686763ba4d277a78ca76f32659220a731/Jinja2-2.10-py2.py3-none-any.whl (126kB)
    100% |################################| 133kB 544kB/s 
Collecting MarkupSafe>=0.23 (from Jinja2>=2.4->flask==0.9)
  Downloading https://files.pythonhosted.org/packages/4d/de/32d741db316d8fdb7680822dd37001ef7a448255de9699ab4bfcbdf4172b/MarkupSafe-1.0.tar.gz
Building wheels for collected packages: flask, MarkupSafe
  Running setup.py bdist_wheel for flask ... done
  Stored in directory: /root/.cache/pip/wheels/69/1b/77/d1aa13cdcc09430d1ba010a48e2cb59fda00f4ef18620ca8dc
  Running setup.py bdist_wheel for MarkupSafe ... done
  Stored in directory: /root/.cache/pip/wheels/33/56/20/ebe49a5c612fffe1c5a632146b16596f9e64676768661e4e46
Successfully built flask MarkupSafe
Installing collected packages: Werkzeug, MarkupSafe, Jinja2, flask
Successfully installed Jinja2-2.10 MarkupSafe-1.0 Werkzeug-0.14.1 flask-0.9
+ Installing Git
Cloning LXC Web Panel...
Cloning into '/srv/lwp'...
remote: Counting objects: 243, done.
remote: Total 243 (delta 0), reused 0 (delta 0), pack-reused 243
Receiving objects: 100% (243/243), 198.33 KiB | 141.00 KiB/s, done.
Resolving deltas: 100% (108/108), done.
Checking connectivity... done.

Installation complete!


Adding /etc/init.d/lwp...
Done
Starting server...done.
Connect you on http://your-ip-address:5000/

```
Once the installation is completed, open your web browser and type the URL http://your-server-ip:5000. 



