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

Now, here is how to use the magic screen command to accomplish what we want, which is access to the volume mountpoint:

```
# Now issue the Magic Screen command and hit <enter> to get a prompt
screen ~/Library/Containers/com.docker.docker/Data/vms/0/tty
# You are now root in the VM, and can issue the following command
ls -l /var/lib/docker/volumes/my-osx-volume
# The directory exists and you will see the actual Mountpoint sub folder "_data"
# Now hit control-a followed by lower case k to kill the screen session
<CTRL-a>k

```

And voila...

```
ls -l /var/lib/docker/volumes/my-os-volume 

```
- Now is a good time to point out that we have created these volumes without ever creating or using a Docker container. This is an indication that a Docker volume is outside of the realm of the normal container-union filesystem.

- Creating Docker Images, that we can also create volumes using a parameter on the container run command, or by adding a VOLUME instruction in the Dockerfile. And, as you might expect, you are able to mount volumes pre-created using the Docker volume create command into containers by using a container run parameter, namely the `--mount` parameter, for example, as follows:

```
# mount a pre-created volume with --mount parameter
docker container run --rm -d \
--mount source=my-vol-02,target=/myvol \
--name vol-demo2 \
volume-demo2:1.0 tail -f /dev/null

```
- This example will run a new container that will mount the existing volume named `my-vol-02`. It will mount that volume in the container at `/myvol`. Note that the preceding example could also have been run without pre-creating the my-vol-02:volume, and the act of running the container with the `--mount` parameter would create the volume as part of the process of starting up the container. Note that any contents defined in the image's mount point folder will be added to the volume when the volume is mounted. However, if a file exists in the image's mount point folder, it also exists in the host's mount point, and the contents of the host's file will be what ends up being in the file. Using an image from this Dockerfile, here is what that looks like:

```
# VOLUME instruction Dockerfile for Docker Quick Start
FROM alpine
RUN mkdir /myvol
RUN echo "Data from image" > /myvol/both-places.txt
CMD ["sh"]


```

Note the Data from image line. Now, using a pre-created volume that contains a file with the matching name of both-places.txt, but has the Data from volume contents in the file, we will run a container based on the image. Here is what happens:

```
# build the demo image
docker image ls
cd volume-demo2
ll
cat Dockerfile
docker image build --rm --tag volume-demo2:1.0 .
docker image ls
cd ..

clear

# mount a pre-created volume with --mount parameter
docker volume create my-vol-02
docker container run --rm -d --mount source=my-vol-02,target=/myvol \
--name vol-demo2 volume-demo2:1.0 tail -f /dev/null

# lets see it
docker container ls
docker volume ls

# show the contents of the file on the local filesysem
sudo cat /var/lib/docker/volumes/my-vol-02/_data/both-places.txt

# show the contents of the file in the contaier
docker container exec -it vol-demo2 cat /myvol/both-places.txt

# they match

# clean up
docker container rm -f $(docker container ls -aq)
docker volume rm -f $(docker volume ls -q)
docker image rm -f volume-demos:latest

```
- As you can see, even though the Dockerfile created a file with the `Data` from image contents, when we ran a container from that image and mounted a volume that had the same file, the contents from the volume (`Data from volume`) prevailed and is what was found in the running container.

- Remember that you cannot mount a pre-created volume via a `VOLUME` instruction in a `Dockerfile`. There is no such thing as a Dockerfile VOLUME instruction named volume. The reason for this is that the Dockerfile cannot dictate the location on the host that a volume is mounted from. Allowing that would be bad for a few reasons. First, since the Dockerfile creates an image, every container that was run from that image would be trying to mount the same host location. That could get real bad real fast. Second, since a container image can be run on different host operating systems, it is quite possible that the definition of the host path for one OS would not even work on another OS. Again, bad. Third, defining the volumes host path would open up all kinds of security holes. Bad, bad, bad! Because of this, running a container from an image build with a Dockerfile that has a VOLUME instruction will always create a new, uniquely-named mount point on the host. Using the VOLUME instruction in a Dockerfile has somewhat limited use cases, such as when a container will run an application that will always need to read or write data that is expected at a specific location in the filesystem but should not be a part of the `Union File System`.

- It is also possible to create a one-to-one mapping of a file on the host to a file in a container. To accomplish this, add a -v parameter to the container run command. You will need to provide the path and filename to the file to be shared from the host and the fully-qualified path to the file in the container. The container run command might look like this:

```
# Map a single file from the host to a container
echo "important data" > /tmp/data-file.txt
docker container run --rm -d \
   -v /tmp/data-file.txt:/myvol/data-file.txt \
   --name vol-demo \
   volume-demo2:1.0 tail -f /dev/null
# Prove it
docker exec vol-demo cat /myvol/data-file.txt


```

There are a few different ways to define the volume in the container run command. To illustrate this point, look at the following run commands, each of which will accomplish the same thing:

```
# Using --mount with source and target
docker container run --rm -d \
   --mount source=my-volume,target=/myvol,readonly \
   --name vol-demo1 \
   volume-demo:latest tail -f /dev/null

# Using --mount with source and destination
docker container run --rm -d \
   --mount source=my-volume,destination=/myvol,readonly \
   --name vol-demo2 \
   volume-demo:latest tail -f /dev/null

# Using -v 
docker container run --rm -d \
   -v my-volume:/myvol:ro \
   --name vol-demo3 \
   volume-demo:latest tail -f /dev/null

```

All three of the preceding container run commands will create a container that has mounted the same volume, in read-only mode. This can be verified with the following command:

```
# Check which container have mounted a volume by name
docker ps -a --filter volume=in-use-volume

```

## Removing volumes

We have already seen and used the volume list command, volume ls, and the inspect command, volume inspect, and I think you should have a good grasp of what these commands do. There are two other commands in the volume-management group, both for volume removal. The first is the volume rm command, which you can use to remove one or more volumes by name. Then, there is the volume prune command; with the prune command, you can remove ALL unused volumes. Be extra careful with the use of this command. Here is the syntax for the remove and prune commands:

```
# Remove volumes command syntax
Usage: docker volume rm [OPTIONS] VOLUME [VOLUME...]
# Prune volumes command syntax
Usage: docker volume prune [OPTIONS]

```

Here are some examples of using the remove and prune commands:

```
# using volumes-from

# start by building the demo image
docker image ls
cd volume-demo2
ll
cat Dockerfile
docker image build --rm --tag volume-demos:latest .
docker image ls
cd ..

clear

# Step 1 - Create the data container
\
docker container run \
--rm -d \
-v data-vol-01:/data/vol1 -v data-vol-02:/data/vol2 \
--name data-container \
volume-demos:latest tail -f /dev/null

# Step 2 - Create the app container that uses the data container
\
docker container run \
--rm -d \
--volumes-from data-container \
--name app-container \
volume-demos:latest tail -f /dev/null

# Prove it
docker container exec app-container ls -l /data

# Prove it more
docker container inspect -f '{{ range .Mounts }}{{ .Name }} {{ end }}' app-container

# clean up

# stop the containers (volumes will still exist)
docker container stop app-container data-container
docker volume ls

# remove volume even though they are still in use (be careful)
docker volume prune --force

# now remove the containers (already gone due to --rm)
docker container rm -f $(docker container ls -aq)

# remove all volumes (should be none left now)
docker volume rm -f $(docker volume ls -q)

# remove the image
docker image rm -f volume-demos:latest

```

Since the in-use-volume volume is mounted in the vol-demo container, it did not get removed with the prune command. You can use a filter on the volume list command to see what volumes are not associated with a container, and as such would be removed with the prune command. Here is the filtered ls command:

```
# Using a filter on the volume ls command
docker volume ls --filter dangling=true

```

## Sharing data between containers with data volume containers

There is another feature of Docker volumes that allows you to share the volume(s) mounted in one Docker container with other containers. It is called data volume containers. Using data volume containers is basically a two-step process. In the first step, you run a container that either creates or mounts Docker volumes (or both), and in the second step, you use the special volume parameter, `--volumes-from`, when running other containers to configure them to mount all of the volumes mounted in the first container. Here is an example:

```
# Step 1
docker container run \
   --rm -d \
   -v data-vol-01:/data/vol1 -v data-vol-02:/data/vol2 \
   --name data-container \
   vol-demo2:1.0 tail -f /dev/null
# Step 2
docker container run \
   --rm -d \
   --volumes-from data-container \
   --name app-container \
   vol-demo2:1.0 tail -f /dev/null
# Prove it
docker container exec app-container ls -l /data
# Prove it more
docker container inspect -f '{{ range .Mounts }}{{ .Name }} {{ end }}' app-container

```

In this example, the first container run command is creating the volumes, but they could have just as easily been pre-created with an earlier container run command, or from a volume create command.
