# Installing LXC on Ubuntu from source

To use the latest version of LXC, you can download the source code from the upstream GitHub repository and compile it:
First, let's install git and clone the repository:
```
root@ubuntu:~# apt-get install git
root@ubuntu:~# cd /usr/src
root@ubuntu:/usr/src# git clone https://github.com/lxc/lxc.git
Cloning into 'lxc'...
remote: Counting objects: 29252, done.
remote: Compressing objects: 100% (156/156), done.
remote: Total 29252 (delta 101), reused 0 (delta 0), 
      pack-reused 29096
Receiving objects: 100% (29252/29252), 11.96 MiB | 12.62 
      MiB/s, done.
Resolving deltas: 100% (21389/21389), done.
root@ubuntu:/usr/src#

```
Next, let's install the build tools and various dependencies:

```
root@ubuntu:/usr/src# apt-get install -y dev-utils 
      build-essential aclocal automake pkg-config git bridge-utils 
      libcap-dev libcgmanager-dev cgmanager
root@ubuntu:/usr/src#


```
Now, generate the configure shell script, which will attempt to guess correct values for different system-dependent variables used during compilation:

```
root@ubuntu:/usr/src# cd lxc
root@ubuntu:/usr/src/lxc#./autogen.sh

```
Its time now to run configure. In this example, I'll enable Linux capabilities and cgmanager, which will manage the cgroups for each container:

```
root@ubuntu:/usr/src/lxc# ./configure --enable-capabilities 
      --enable-cgmanager
...
----------------------------
Environment:
   - compiler: gcc
   - distribution: ubuntu
   - init script type(s): upstart,systemd
   - rpath: no
   - GnuTLS: no
   - Bash integration: yes
Security features:
   - Apparmor: no
   - Linux capabilities: yes
   - seccomp: no
   - SELinux: no
   - cgmanager: yes
Bindings:
   - lua: no
   - python3: no
Documentation:
   - examples: yes
   - API documentation: no
   - user documentation: no
Debugging:
   - tests: no
   - mutex debugging: no
Paths:
Logs in configpath: no
root@ubuntu:/usr/src/lxc#

```
From the preceding abbreviated output we can see what options are going to be available after compilation. 
Notice that we are not enabling any of the security features for now, such as Apparmor.
Next, compile with make:
```
root@ubuntu:/usr/src/lxc# make

```
Finally, install the binaries, libraries, and templates:

```
root@ubuntu:/usr/src/lxc# make install

```
As of this writing, the LXC binaries look for their libraries in a different path than where they were installed. To fix this just copy them to the correct location:

```
root@ubuntu:/usr/src/lxc# cp /usr/local/lib/liblxc.so* 
      /usr/lib/x86_64-linux-gnu/


```
To check the version that was compiled and installed, execute the following code:

```

root@ubuntu:/usr/src/lxc# lxc-create --version
4.0.0
root@ubuntu:/usr/src/lxc#  

```

