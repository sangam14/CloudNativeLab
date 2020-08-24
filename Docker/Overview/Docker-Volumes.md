---
layout: default
title: Docker Volumes
parent: Docker Fundamental
nav_order: 4
---

# What is a Docker volume?

- Creating Docker Images, Docker uses a special filesystem called a Union File System. This is the key to Docker's layered image model and 
allows for many of the features that make using Docker so desirable. However, the one thing that the Union File System does not provide for 
is the persistent storage of data. If you recall, the layers of a Docker image are read-only. 

- When you run a container from a Docker image, the Docker daemon creates a new read-write layer that holds all of the live data that represents your container. When your container makes changes 
to its filesystem, those changes go into that read-write layer. As such, when your container goes away, taking the read-write layer goes with it,
and any and all changes the container made to data within that layer are deleted and gone forever. That equals non-persistent storage.

- Remember, however,that generally speaking this is a good thing. A great thing, in fact. Most of the time, this is exactly what we want to happen. 
Containers are meant to be ephemeral and their state data is too. However, there are plenty of use cases for persistent data, such as customer 
order data for a shopping site. It would be a pretty bad design if all the order data went bye-bye if a container crashed or had to be re-stacked. 

- Enter the Docker volume. The Docker volume is a storage location that is completely outside of the Union File System. As such,  it is not bound by the same rules that are placed on the read-only layers of an image or the read-write layer of a container.
A Docker volume is a storage location that, by default, is on the host that is running the container that uses the volume.  When the container goes away, either by design or by a catastrophic event, the Docker volume stays behind and 
is available to use by other containers. The Docker volume can be used by more than one container at the same time.

- The simplest way to describe a Docker volume is this: a Docker volume is a folder that exists on the Docker host and is mounted and accessible inside a 
running Docker container. The accessibility goes both ways, allowing the contents of that folder to be modified from inside the container, or on the Docker
host where the folder lives.

- Now, this description is a bit of a generalization. Using different volume drivers, the actual location of the folder being mounted as a volume can be hosted
somewhere not on the Docker host. With volume drivers, you are able to create your volumes on remote hosts or cloud providers. For example, you can use an 
NFS driver to allow the creation of Docker volumes on a remote NFS server.

- Like Docker image and Docker container, the volume commands represent their own management category. As you would expect, the top-level management command for 
volumes is as follows:

```
# Docker volume managment command
docker volume

```

The subcommands available in the volume management group include the following:

```
# Docker volume management subcommands
docker volume create         # Create a volume
docker volume inspect        # Display information on one or more volumes
docker volume ls             # List volumes
docker volume rm             # Remove one or more volumes
docker volume prune          # Remove all unused local volumes

```

## Creating Docker volumes


There are a few ways to create a Docker volume. One way is to use the volume create command. The syntax for that command is as follows:

```
# Syntax for the volume create command
Usage:docker volume create [OPTIONS] [VOLUME]
```

In addition to the optional volume name parameter, the create command allows for these options:

```
# The options available to the volume create command:
-d, --driver string         # Specify volume driver name (default "local")
--label list                # Set metadata for a volume
-o, --opt map               # Set driver specific options (default map[])

```
Let's start with the simplest example:

```
# Using the volume create command with no optional parameters
docker volume create

```

Executing the preceding command will create a new Docker volume and assign it a random name. The volume will be created using the built-in local driver (by default).
Using the volume ls command, you can see what random name the Docker daemon assigned our new volume. It will look something like this:

```
docker volume create 
docker volume ls 
```

Stepping it up a notch, let's create another volume, this time supplying an optional volume name with the command. The command will look something like this:

```
# Create a volume with a fancy name
docker volume create my-vol-02

```
This time, the volume is created and is given the name my-vol-02, as requested

This volume still uses the default local driver. Using the local driver simply means that the actual location for the folder this volume represents can be found locally on the Docker host. 
We can use the volume inspect subcommand to see where that folder can actually be found:

```
docker volume inspect my-vol-02 
```
- As you can see in the preceding screenshot, the volume's mount point is on the Docker host's filesystem at `/var/lib/docker/volumes/my-vol-02/_data`. Notice 
that the folder path is owned by root, which means you need elevated permissions to access the location from the host. Notice also that this example was run on a Linux host.

- If you are using OS X, you need to remember that your Docker install is actually using a mostly seamless virtual machine. One of the areas where the 
seams do show up is with the use of the Docker volumes. The mount point that is created when you create a Docker volume on an OS X host is stored in the filesystem of the virtual machine, not on your OS X filesystem. When you use the docker volume inspect command and see the path to the mount point of your volume, it is not a path on your OS X filesystem, but rather the path on the filesystem of the hidden virtual machine.

- There is a way to view the filesystem (and other features) of that hidden virtual machine. With a command, often referred to as the 
Magic Screen command, you can access the running Docker VM. That command looks like this:

```
# The Magic Screen command
screen ~/Library/Containers/com.docker.docker/Data
/com.docker.driver.amd64-linux/tty
# or if you are using Mac OS High Sierra
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty

```
Note : Use `Ctrl + AK` to kill the screen session. You can detach with `Ctrl + A Ctrl + D`, then use `screen -r` to reconnect, but don't detach and then start a new screen session. Running more than one screen to the VM will give you tty garbage.

Here is an example of accessing the mount point for a volume created on an OS X host. Here is the setup:

```
# Start by creating a new volume
docker volume create my-osx-volume
# Now find the Mountpoint
docker volume inspect my-osx-volume -f "{{json .Mountpoint}}"
# Try to view the contents of the Mountpoint's folder
sudo ls -l /var/lib/docker/volumes/my-osx-volume
# "No such file or directory" because the directory does not exist on the OS X host

```
