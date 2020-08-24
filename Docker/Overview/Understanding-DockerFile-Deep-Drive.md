---
layout: default
title: Understanding DockerFile Deep Drive 
parent: Docker Fundamental
nav_order: 3
---

# Describe the use of Dockerfile. 


- A Dockerfile is a text document that contains all the commands a user could call on the command line to assemble an image.
Think of it as a shellscript. It gathered multiple commands into a single document to fulfill a single task.


## build command is used to create an image from the Dockerfile.

```
 $ docker build 
```
## You can name your image as well.
```
 $ docker build -t my-image 
```
## If your Dockerfile is placed in another path,
```
 $ docker build -f /path/to/a/Dockerfile . 
 ```


# DockerFile In Depth with all Instruction 

## The `FROM `instruction

- Every `Dockerfile` must have a `FROM` instruction, and it must be the first instruction in the file.  

The `FROM` instruction sets the base for the image being created and instructs the Docker daemon that the base of the new image should be the existing Docker image specified as the parameter. The specified image can be described using the same syntax we saw in the Docker container run command from Chapter 2, Learning Docker Commands. Here, it's a `FROM` instruction that specifies using the official nginx image with a version of 1.15.2:

```
# Dockerfile
FROM nginx:1.15.2

```
- Note that in this example, there is no repository specified that indicates that the specified image is the official nginx image. If no tag is specified, the latest tag will be assumed.

- The `FROM` instruction will create the first layer in our new image. That layer will be the size of the image specified in the instruction's parameter so it is best to specify the smallest image that meets the criteria needed for your new image. An application-specific image, such as nginx, is going to be smaller than an OS image, such as ubuntu. And, the OS image for alpine will be much smaller than images of other OSes, such as Ubuntu, CentOS, or RHEL. There is a special keyword that can be used as the parameter to the FROM instruction. It is scratch. Scratch is not an image that you can pull or run, it just a signal to the Docker daemon that you want to build an image with an empty base-image layer. The `FROM` scratch instruction is used as the base layer for many other base images, or for specialized app-specific images. You have already seen an example of such a specialized app image: hello-world. The full Dockerfile for the hello-world image looks like this:

```
# hello-world Dockerfile
FROM scratch
COPY hello /
CMD ["/hello"]


```

We will discuss the `COPY` and `CMD` instructions shortly, but you should get a sense of how small the hello-world image is based on its Dockerfile. In the world of Docker images, smaller is definitely better. 


# The `LABEL` instruction

- The `LABEL` instruction is a way to add metadata to your Docker image. This instruction adds embedded key-value pairs to the image. The `LABEL` instruction adds a zero-byte-sized layer to the image when it is created. An image can have more than one LABEL, and each `LABEL` instruction can provide one or more LABELs. The most common use for the LABEL instruction is to provide information about the image maintainer. This data used to have its own instruction. See the following tip box about the now-deprecated `MAINTAINER` instruction. Here are some examples of valid `LABEL` instructions:

```
# LABEL instruction syntax
# LABEL <key>=<value> <key>=<value> <key>=<value> ...
LABEL maintainer="sangam biradar <sangambiradar@hotmail.com>"
LABEL "description"="My development Ubuntu image"
LABEL version="1.0"
LABEL label1="value1" \
 label2="value2" \
 lable3="value3"
LABEL my-multi-line-label="Labels can span \
more than one line in a Dockerfile."
LABEL support-email="support@something.com" support-phone="123-456-7890"


```

- The `LABEL` instruction is one of the instructions that can be used multiple times in a Dockerfile. You will learn later that some instructions that can be used multiple times will result in only the last use being significant, thus ignoring all previous uses. The LABEL instruction is different. Every use of the `LABEL` instruction adds an additional label to the resulting image. However, if two or more uses of `LABEL` have the same key, the label will get the value provided in the last matching `LABEL` instruction. That looks like this:

```
# earlier in the Dockerfile
LABEL version="1.0"
# later in the Dockerfile...
LABEL version="2.0"
# The Docker image metadata will show version="2.0"

```
It is important to know that the base image specified in your `FROM` instruction may include labels created with the LABEL instruction and that they will automatically be included in the metadata of the image you are building. If a `LABEL` instruction in your Dockerfile uses the same key as a `LABEL` instruction used in the FROM image's Dockerfile, your (later) value will override the one in the `FROM` image. You can view all of the labels for an image by using the inspect command:

```
docker inspect --format `{{json .Config}}' < dockerimage_name : tag > | jq `.Labels'

```

Note

The `MAINTAINER` instructionThere is a Dockerfile instruction specifically for providing the info about the image maintainer, however, this instruction has been deprecated. Still, you will probably see it used in a Dockerfile at some point. The syntax goes like this: `"maintainer": "sangam biradar <sangambiradar@hotmail.com>"`.


## The `COPY` instruction 

- You have already seen an example of using the `COPY` instruction in the `hello-world` Dockerfile shown in The `FROM` instruction section. The `COPY `instruction is used to copy files and folders into the Docker image being built. The syntax for the COPY instruction is as follows:

```
# COPY instruction syntax
COPY [--chown=<user>:<group>] <src>... <dest>
# Use double quotes for paths containing whitespace)
COPY [--chown=<user>:<group>] ["<src>",... "<dest>"]

```

- Note that the `--chown `parameter is only valid for Linux-based containers. Without the `--chown` parameter, the owner `ID` and group `ID` will both be set to` 0`.

- The `<src> `or source is a filename or folder path and is interpreted to be relative to the context of the build. We will talk more about the build context later in this chapter, but for now, think of it as where the build command is run. The source may include wildcards.

- The `<dest>` or destination is a filename or path inside of the image being created. The destination is relative to the root of the image's filesystem unless there is a preceding `WORKDIR` instruction. We will discuss the WORKDIR instruction later, but for now, just think of it as a way to set the current working directory. When the COPY command comes after a `WORKDIR `instruction in a Dockerfile, the file or folders being copied into the image will be placed in the destination relative to the current working directory. If the destination includes a path with one or more folders, all of the folders will be created if they don't already exist.

- In our earlier hello-world Dockerfile example, you saw a `COPY` instruction that copied an executable file, named hello, into the image at the filesystem's root location. It looked like this: `COPY hello /`. That is about as basic a COPY instruction as can be used. Here are some other examples:

```

# COPY instruction Dockerfile for Docker Quick Start
FROM alpine:latest
LABEL maintainer=" sangam biradar "
LABEL version=1.0
# copy multiple files, creating the path "/theqsg/files" in the process
COPY file* theqsg/files/
# copy all of the contents of folder "folder1" to "/theqsg/" 
# (but not the folder "folder1" itself)
COPY folder1 theqsg/
# change the current working directory in the image to "/theqsg"
WORKDIR theqsg
# copy the file special1 into "/theqsg/special-files/"
COPY --chown=35:35 special1 special-files/
# return the current working directory to "/"
WORKDIR /
CMD ["sh"]

```

We can see what the resulting image's filesystem would get using the preceding Dockerfile by running a container from the image, and executing an ls command, which would look like this:

```
docker container run --rm containerlab-demo:1.0  ls -l theqsg

```
you can see that folders specified in the destination path were created during the COPY. You will also notice that providing the `--chown `parameter sets the owner and group on the destination files. An important distinction is that when the source is a folder, the contents of the folder are copied but not the folder itself. Notice that using a `WORKDIR` instruction changes the path in the image filesystem and following COPY instructions will now be relative to the new current working directory. In this example, we returned the current working directory to `/ `so that commands executed in containers will run relative to `/`.


## The ADD instruction

- The `ADD` instruction is used to copy files and folders into the Docker image being built. The syntax for the `ADD` instruction is as follows:

```
# ADD instruction syntax
ADD [--chown=<user>:<group>] <src>... <dest>
# Use double quotes for paths containing whitespace)
ADD [--chown=<user>:<group>] ["<src>",... "<dest>"]

```
About now, you are thinking that the `ADD` instruction seems to be just like the `COPY` instruction that we just reviewed. Well, you are not wrong. Basically, all of the things we saw the `COPY` instruction do, the ADD instruction can do as well. It uses the same syntax as the COPY instruction and the effects of `WORKDIR` instructions are the same between the two. So, why do we have two commands that do the same thing?

## The difference between COPY and ADD

- The answer is that the `ADD` instruction can actually do more than the `COPY` instruction. The more is dependent upon the values used for the source input. With the `COPY` instruction, the source can be files or folders. However, with the `ADD` instruction, the source can be files, folders, a local .tar file, or a `URL`.

- When the `ADD` instruction has a source value that is a `.tar` file, the contents of that `TAR` file are extracted into a corresponding folder inside the image.

- Note
    - When you use a `.tar` file as the source in an ADD instruction and include the` --chown` parameter, you might expect the owner and group in the image to be set on the files extracted from the archive. This is currently not the way it works. Unfortunately, the owner, group, and permissions on the extracted contents will match what is contained within the archive in spite of the `--chown` parameter. When you use a .tar file, you will probably want to include `RUN chown -R X:X` after the `ADD`.
    
- As mentioned, the `ADD` instruction can use a URL as the source value. Here is an example Dockerfile that includes an ADD instruction using a URL:    

```
# ADD instruction Dockerfile for Docker Quick Start
FROM alpine
LABEL maintainer="sangam biradar <sangambiradar@hotmail.com>"
LABEL version=3.0
ADD https://github.com/docker-library/hello-world/raw/master/amd64/hello-world/hello /
RUN chmod +x /hello
CMD ["/hello"]

```

While using a URL in an `ADD `instruction works, downloading the file into the image, this feature is not recommended, even by Docker. Here is what the Docker documentation has to say about using `ADD`:

![](https://raw.githubusercontent.com/sangam14/ContainerLabs/master/img/dockerfile-doc.png)

So, generally speaking, whenever you can get the desired content into the image using a COPY instruction, then you should choose to use `COPY` instead of `ADD`.


## The `ENV` instruction

- As you may guess, the `ENV` instruction is used to define environment variables that will be set in the running containers created from the image being built.
The variables are defined using typical key-value pairs. A Dockerfile can have one or more ENV instructions. Here is the `ENV` instruction syntax:

```
# ENV instruction syntax
# This is the form to create a single environment variable per instruction
# Everything after the space following the <key> becomes the value
ENV <key> <value>
# This is the form to use when you want to create more than one variable per instruction
ENV <key>=<value> ...

```

Each `ENV` instruction will create one or more environment variables (unless the key name is repeated). Let's take a look at some ENV instructions in a Dockerfile:

```
# ENV instruction Dockerfile for Docker Quick Start
FROM alpine
LABEL maintainer="sangam birdar <sangambiradar@hotmail.com>"
ENV appDescription This app is a sample of using ENV instructions
ENV appName=env-demo
ENV note1="The First Note First" note2=The\ Second\ Note\ Second \
note3="The Third Note Third"
ENV changeMe="Old Value"
CMD ["sh"]

```

After building the image using this Dockerfile, you can inspect the image metadata and see the environment variables that have been created:

```
docker inspect --format `{{json .Config}}' < dockerimage_name : tag > | jq `.Env'


```

Environment variables can be set (or overridden) when a container is run using the `--env` parameter. Here, we see this feature in action:

```
docker container run -rm --env chnageMe="new value" --env adhoc="run time"  sangam14-env-demo env

```

- It is important to know that using `ENV` instructions create a zero-byte-sized additional layer in the resulting image. If you are adding more than one environment variable to your image and can use the form of the instruction that supports setting multiple variables with one instruction, doing so will only create a single additional image layer, so that is the way to go.


## The `ARG` instruction

Sometimes when building Docker images, you may need to use variable data to customize the build. The ARG instruction is the tool to handle that situation. To use it, you add ARG instructions to your Dockerfile, and then when you execute the build command, you pass in the variable data with a --build-arg parameter. The `--build-arg` parameter uses the now familiar key-value pair format:

```
# The ARG instruction syntax
ARG <varname>[=<default value>]

# The build-arg parameter syntax
docker image build --build-arg <varname>[=<value>] ...

```
- You can use multiple `ARG` instructions in your Dockerfile with corresponding `--build-arg` parameters on the docker image build commands. You have to include an ARG instruction for every use of the --build-arg parameter. Without the ARG instruction, the `--build-arg` parameter will not be set during the build, and you will get a warning message. If you do not provide a `--build-arg` parameter or you do not provide the value part of the key-value pair for a `--build-arg ` parameter for an existing `ARG` instruction, and that ARG instruction includes a default value, then the variable will be assigned the default value.


- Be aware that during the image build, even though `--build-arg` is included as a parameter of the docker image build command, the corresponding variable does not get set until the `ARG `instruction is reached in the Dockerfile. Said another way, the value of the key-value pair of a `--build-arg` parameter will never be set until after its corresponding ARG line in the Dockerfile.


```
# ARG instruction Dockerfile for Docker Quick Start
FROM alpine
LABEL maintainer="sangam biradar"

ENV key1="ENV is stronger than an ARG"
RUN echo ${key1}
ARG key1="not going to matter"
RUN echo ${key1}

RUN echo ${key2}
ARG key2="defaultValue"
RUN echo ${key2}
ENV key2="ENV value takes over"
RUN echo ${key2}
CMD ["sh"]


```

Create a Dockerfile with the contents shown in the preceding code block and run the following build command to see how the scope of the `ENV `and `ARG` instructions play out:

```

# Build the image and look at the output from the echo commands
docker image build --rm \
 --build-arg key1="buildTimeValue" \
 --build-arg key2="good till env instruction" \
 --tag arg-demo:2.0 .

```
- You will see by the first ` echo ${key1} `that even though there is a `--build-arg ` parameter for key1, it will not be stored as `key1` because there is an `ENV` instruction that has the same key name. This still holds true for the second echo `${key1}`, which is after the ARG key1 instruction. The `ENV` variable values will always be the winner when there are both ARG and EVN instructions with the same key name.

- Then, you will see that the first `echo ${key2}` is empty even though there is a `--build-arg` parameter for it. It is empty because we have not reached the ARG `key2 `instruction yet. The second echo `${key2}` will contain the value from the corresponding `--build-arg` parameter even though there is a default value provided in the `ARG key2` instruction. The final echo `${key2}` will show the value provided in the `ENV` key2 instruction in spite of there being both a default value in the `ARG` and a value passed in via the `--build-arg` parameter. Again, this is because ENV always trumps `ARG`.


## The difference between `ENV` and `ARG`

Again, here is a pair of instructions that have a similar functionality. They both can be used during the build of an image, setting parameters to be available to use within other Dockerfile instructions. The other Dockerfile instructions that can use these parameters are `FROM, LABEL, COPY, ADD, ENV, USER, WORKDIR, RUN, VOLUME, EXPOSE, STOPSIGNAL, and ONBUILD`. Here is an example of using the ARG and ENV variables in other Docker commands:

```
# ENV vs ARG instruction Dockerfile for Docker Quick Start
FROM alpine
LABEL maintainer="sangam biradar "
ENV lifecycle="production"
RUN echo ${lifecycle}
ARG username="35"
RUN echo ${username}
ARG appdir
RUN echo ${appdir}
ADD hello /${appdir}/
RUN chown -R ${username}:${username} ${appdir}
WORKDIR ${appdir}
USER ${username}
CMD ["./hello"]

```
With this Dockerfile, you would want to provide `--build-arg` parameters for the appdirARG instruction, and the username (if you want to override the default) to the build command. You could also provide an `--env` parameter at runtime to override the lifecycle variable. Here are possible build and run commands you could use:

```
# Build the arg3 demo image
docker image build --rm \
   --build-arg appdir="/opt/hello" \
   --tag arg-demo:3.0 .

# Run the arg3 demo container
docker container run --rm --env lifecycle="test" arg-demo:3.0
```
While the `ENV` and `ARG` instructions might seem similar, they are actually quite different. Here are the key differences to remember between the parameters created by the ENV and `ARG` instructions:
  - ENVs persist into running containers, ARGs do not.
  - ARGs use corresponding build parameters, ENVs do not.
  - `ENV` instructions must include both a key and a value, ARG instructions have a key but the (default) value is optional.
  - ENVs are more significant than ARGs

Note :- You should never use either `ENV` or `ARG` instructions to provide secret data to the build command or resulting containers because the values are always visible in clear text to any user that runs the docker history command.

## The `USER` instruction

The `USER` instruction allows you to set the current user (and group) for all of the instructions that follow in the Dockerfile, and for the containers that are run from the built image. The syntax for the `USER` instruction is as follows:

```
# User instruction syntax
USER <user>[:<group>] or
USER <UID>[:<GID>]

```

If a named user (or group) is provided as parameters to the USER instruction, that user (and group) must already exist in the passwd file (or group file) of the system, or a build error will occur. If you provide the UID (or GID) as the parameter to the USER command, the check to see whether the user (or group) exists is not performed. Consider the following Dockerfile:

```
# USER instruction Dockerfile for Docker Quick Start 
FROM alpine
LABEL maintainer="sangam biradar"
RUN id
USER games:games
run id
CMD ["sh"]


```

When the image build starts, the current user is root or `UID=0GID=0`. Then, the USER instruction is executed to set the current user and group to `games:games`. Since this is the last use of the USER instruction in the Dockerfile, all containers run using the built image will have the current user (and group) set to games. Here is what the build and run look like:

```
docker image build --rm --tag sangam14-user-demo .

```
 
 Notice that the output from `Step 3/6:RUN` id shows the current user as root, and then in `Step 5/6` (which is after the USER instruction) it shows the current user as games. Finally, notice that the container run from the image has the current user games. The `USER` instruction creates a zero-byte-sized layer in the image.
 
 
## The `WORKDIR` instruction

We have seen the `WORKDIR` instruction used in some of the examples used to demonstrate other instructions. It is sort of like a combination of the Linux cd and mkdir commands. The `WORKDIR` instruction will change the current working directory in the image to the value provided in the instruction. If any segment of the path in the parameter of the `WORKDIR` instruction does not yet exist, it will be created as part of the execution of the instruction. The syntax for the `WORKDIR` instruction is as follows:

```
# WORKDIR instruction syntax
WORKDIR instruction syntax
WORKDIR /path/to/workdir

```
The `WORKDIR` instruction can use `ENV` or `ARG` parameter values for all or part of its parameter. A Dockerfile can have more than one `WORKDIR` instruction, and each subsequent WORKDIR instruction will be relative to the previous one (if a relative path is used). Here is an example that demonstrates this possibility:

```
# WORKDIR instruction Dockerfile for Docker Quick Start
FROM alpine
# Absolute path...
WORKDIR /
# relative path, relative to previous WORKDIR instruction
# creates new folder
WORKDIR sub-folder-level-1
RUN touch file1.txt
# relative path, relative to previous WORKDIR instruction
# creates new folder
WORKDIR sub-folder-level-2
RUN touch file2.txt
# relative path, relative to previous WORKDIR instruction
# creates new folder
WORKDIR sub-folder-level-3
RUN touch file3.txt
# Absolute path, creates three sub folders...
WORKDIR /l1/l2/l3
CMD ["sh"]

```

Building the image from this Dockerfile will result in the image having three levels of nested folders. Running a container from the image and listing the files and folders will look like this:

```
docker container run --rm sangam14-workdir-demo  ls -lR /sub-folder-1

```

The `WORKDIR` instruction will create a zero-byte-sized layer in the resulting image.


## The `VOLUME` instruction

- You should remember that a Docker image is made up of a series of read-only layers built upon one another, and that when you run a container from a Docker image, it creates a new read-write layer that you can think of as being on top of the read-only layers. All the changes to the container are applied to the read-write layer. If you make a change to a file found in one of the read-only layers, a copy of that file is made and added to the read-write layer. Then, all the changes are applied to the copy. The copy hides the version found in the read-only layer so, from the point of view of the running container, there is only one version of the file, and it is the one that has been changed. This is roughly how the Unified File System works. 

- This is actually a great thing. However, it presents a challenge, this being that when the running container exits and is removed, all of the changes are removed with it. This is normally OK until you want to have some data that persists after the life of the container, or when you want to share data between containers. Docker has an instruction to help you solve this issue, the `VOLUME` instruction.

- The `VOLUME` instruction will create a storage location that is outside of the United File System, and by so doing, allow storage to persist beyond the life of your container. Here is the syntax of the `VOLUME` instruction:

```
# VOLUME instruction syntax
VOLUME ["/data"]
# or for creating multiple volumes with a single instruction
VOLUME /var/log /var/db /moreData

```
- Other ways to create volumes are to add volume parameters to the docker container run command or to use the docker volume create command. 

- Here is a simple example Dockerfile. It creates a volume at `/myvol` that will have a file named `greeting`:

```
# VOLUME instruction Dockerfile for Docker Quick Start
FROM alpine
RUN mkdir /myvol
RUN echo "hello world" > /myvol/greeting
VOLUME /myvol
CMD ["sh"]
```
Running a container based on an image made from this Dockerfile will create a mount point on the host system that initially contains the greeting file. When the container exits, the mount point will remain. Be careful with the use of the --rm parameter when running a container that has mount points you wish to persist. Using --rm, with no other volume parameters, will cause the mount points to be cleaned up along with the container when it exits. Here is what that looks like:

```
docker image ls
cd volume-demo
ll
cat Dockerfile
docker image build --rm --tag volume-demo:1.0 .
docker image ls
clear
docker container run --rm -it --mount source=myvolsrc,target=/myvol sangam14-volume-demo:1.0
clear
docker volume ls
docker container run --rm -d --name vol-demo volume-demo:1.0 tail -f /dev/null
docker volume ls
docker container stop vol-demo
docker container ls
docker volume ls
clear

```

We start out with no volumes. Then, we run a container based on the image made from the preceding Dockerfile in detached mode. We check the volumes again, and we see the volume created by running the container. Then, we stop the container and check for volumes again, and the volume is now gone. Usually, the purpose of using a VOLUME instruction is to have data in a mount point that persists after the container is gone. So, if you are going to use --rm when you run a container, you should include the `--mount` run parameter


```
docker volume create myvolsrc
docker container run -d --name vol-demo --mount source=myvolsrc,target=/myvol sangam14-volume-demo:1.0 tail -f /dev/null
docker container exec vol-demo ls -l /myvol
docker volume inspect myvolsrc -f "{{.Mountpoint}}"
echo /var/lib/docker/volumes/myvolsrc/_data/new-file.txt
touch /var/lib/docker/volumes/myvolsrc/_data/new-file.txt\n
sudo touch /var/lib/docker/volumes/myvolsrc/_data/new-file.txt\n
docker container exec vol-demo ls -l /myvol
docker container stop vol-demo
docker container rm vol-demo
docker volume rm myvolsrc
docker volume ls
docker container ls

```
- In this demo, we run a container that is based on an image created with the preceding Dockerfile. Then, we list the volumes and see the myvolsrc volume (we already knew the name since we provided it in the run command, but you can use the ls command to find volume names that you might not otherwise know). Using the volume's name, we inspect the volume to find its mount point on the host. To verify the contents of the volume in the container, we use an exec command to do an ls of the folder. Next, using the mount point path, we create a new file using the touch command. Finally, we use the same exec command and see that inside the container the volume has been changed (from actions outside of the container). Similarly, if the container makes changes to the contents of the volume, they are reflected instantly on the host mount point.

- Using the VOLUME instruction is both powerful and dangerous. It is powerful in that it lets you have data that will persist beyond the life of your containers. It is dangerous because data is passed instantaneously from the container to the host, and if the container is ever compromised, that can spell trouble. That is why, for security purposes, it is best practice to not include host-based VOLUME mounts in your Dockerfile

- The `VOLUME` instruction will add a zero-bytes sized layer to your resulting Docker image.


## The `EXPOSE` instruction

The `EXPOSE` instruction is a way to document what network ports the image expects to be opened when a container is run from the image built using the Dockerfile. The syntax for the EXPOSE instruction is as follows:

```
# EXPOSE instruction syntax
EXPOSE <port> [<port>/<protocol>...]


```
- it is important to understand that including the EXPOSE instruction in the Dockerfile does not actually open network ports in containers. When containers are run from the images with the EXPOSE instruction in their Dockerfile, it is still necessary to include the `-p` or `-P `parameters to actually open the network ports to the container.

- You can include multiple EXPOSE instructions in your Dockerfile as needed. Including the -P parameter at runtime is a shortcut way to automatically open ports for all of the EXPOSE instructions included in the Dockerfile. The corresponding host ports will be randomly assigned when using the `-P` parameter on the run command.

- Think of the `EXPOSE` instruction as a message from the image developer telling you that the application in the image is expecting you to open the indicated port(s) when you run your containers. The `EXPOSE` instruction creates a zero-byte-sized layer in the resulting image.


## The `RUN` instruction

The `RUN` instruction is the real workhorse of the Dockerfile. It is the tool by which you affect the most change in the resulting docker image. Basically, it allows you to execute any command in the image. There are two forms of the RUN instruction. Here is the syntax:

```
# RUN instruction syntax
# Shell form to run the command in a shell
# For Linux the default is "/bin/sh -c"
# For Windows the default is "cmd /S /C"
RUN <command>

# Exec form
RUN ["executable", "param1", "param2"]

```

- Every `RUN` instruction creates a new layer in the image, and the layers for each instruction that follow will be built on the results of the `RUN` instruction's layer. The shell form of the instruction will use the default shell unless it is overridden using a SHELL instruction, which we will discuss in The `SHELL` instruction section. If you are building a container that does not include a shell, you will need to use the exec form of the `RUN` instruction. You can also use the exec form of the instruction to use a different shell. For example, to run a command using the bash shell, you could add a `RUN` instruction, like so:

```
# Exec form of RUN instruction using bash
RUN ["/bin/bash", "-c", "echo hello world > /myvol/greeting"]

```
The uses for the RUN command are limited only by the imagination, so providing an exhaustive list of RUN instruction samples would be impossible, but here are a few using both forms of the instruction, just to give you some ideas:

```
# RUN instruction Dockerfile for Docker Quick Start
FROM ubuntu
RUN useradd --create-home -m -s /bin/bash dev
RUN mkdir /myvol
RUN echo "hello DQS Guide" > /myvol/greeting
RUN ["chmod", "664", "/myvol/greeting"]
RUN ["chown", "dev:dev", "/myvol/greeting"]
VOLUME /myvol
USER dev
CMD ["/bin/bash"]

```

You can use the following code to create a custom prompt displayed when you shell into your containers. If you don't like the whale graphic, you can switch it up and use anything you like better. I've included some of my favorite options. Here's the code:

The resulting prompt looks like this:

```
docker container run -rm -it sangam-run-demo 

```

## The `CMD` instruction

- The `CMD` instruction is used to define the default action taken when containers are run from images built with their Dockerfile. While it is possible to include more than one `CMD` instruction in a Dockerfile, only the last one will be significant. Essentially, the final `CMD` instruction provides the default action for the image. This allows you to either override or use the CMD in the image used in the `FROM` instruction of your Dockerfile. Here is an example where a trivial Dockerfile does not contain a `CMD` instruction and relies on the one found in the ubuntu image used in the `FROM` instruction:

```
docker history ubuntu 

cat Dockerfile 
From ubuntu 

```
- You can see from the output of the history command that the ubuntu image includes the `CMD ["/bin/bash"]` instruction. You will also see that our Dockerfile does not have its own CMD instruction. When we run the container, the default action is to run "/bin/bash".

- There are three forms of the CMD instruction. The first is a shell form. The second is an exec form, which is the best practice form to use. And, the third is a special exec form that has exactly two parameters, and it is used in conjunction with the `ENTRYPOINT` instruction, which we will talk about in The `ENTRYPOINT` instruction section. Here is the syntax for the `CMD` instruction.

```
# CMD instruction syntax
CMD command param1 param2 (shell form)
CMD ["executable","param1","param2"] (exec form)
CMD ["param1","param2"] (as default parameters to ENTRYPOINT)

```

Here are a few `CMD` instruction examples for your enjoyment:

```
# CMD instruction examples
CMD ["/bin/bash"]
CMD while true; do echo 'DQS Expose Demo' | nc -l -p 80; done
CMD echo "How many words are in this echo command" | wc -
CMD tail -f /dev/null
CMD ["-latr", "/var/opt"]

```
- Like the `RUN` instruction, the shell form of the `CMD` instruction will use the `["/bin/sh", "-c"]` shell command (or `["cmd", "/S", "/C"]` for Windows) by default unless it is overridden with a `SHELL` instruction. However, unlike the `RUN` instruction, the CMD instruction does not execute anything during the building of the image but instead is executed when containers built from the image are run. If the container image being built will not have a shell, then the exec form of the instruction can be used as it does not invoke a shell. The `CMD` instruction adds a zero-byte-sized layer to the image.

## The `ENTRYPOINT `instruction

```
# ENTRYPOINT instruction Dockerfile for Docker Quick Start
FROM alpine
RUN apk add curl
ENTRYPOINT ["curl"]
CMD ["--help"]

```

We can run the container image with no overriding CMD parameter and it will show help for the curl command. However, when we run the container with a CMD override parameter, in this case, a URL, the response will be to curl the URL. Take a look:

```
docker container run sangam14-entrypoint-demo google.com
```
When run parameters are provided to a container that has the exec form of the ENTRYPOINT command, those parameters will be appended to the ENTRYPOINT instruction, overriding anything provided in a CMD instruction. In this example, `--help` is overridden with the `google.com` run parameter, so the resulting instruction is curl `google.com`. Here is the actual syntax for the `ENTRYPOINT `instruction:

```
# ENTRYPOINT instruction syntax
ENTRYPOINT command param1 param2 (shell form)
ENTRYPOINT ["executable", "param1", "param2"] (exec form, best practice)

```
- Like the `CMD` instruction, only the last `ENTRYPOINT` instruction is significant. Again, this allows you to either use or override the ENTRYPOINT instruction in the FROM image used. Like both the `RUN` and `CMD` instructions, using the shell form will invoke a shell as `["/bin/sh", "-c"]` (or` ["cmd", "/S", "/C"]` on Windows). This is not the case when using the exec form of the instruction. This is key if you have an image that does not have a shell or if the shell is not available to the active user context. However, you will not get shell processing, so any shell environment variables will not get substituted when using the exec form of the instruction. It is generally considered best practice to use the exec form of the `ENTRYPOINT` instruction whenever possible.


## The difference between CMD and ENTRYPOINT

- Here again, we have two instructions that on the surface seem to be very much the same. It is true that there is some overlap of functionality between the two. Both instructions provide a way to define a default application that is executed when containers are run. However, they each serve their own unique purpose, and in some cases work together to provide greater functionality than either instruction alon


- The best practice is to use the ENTRYPOINT instruction when you want a container to execute as an application, providing a specific (developer) defined function, and to use CMD when you want to give the user more flexibility in what function the container will serve.

- Both of these instructions have two forms: a shell form and an exec form. It is best practice to use the exec form of either whenever possible. The reason for this is that the shell form, by definition, will run `["/bin/sh", "-c"]` (or `["cmd", "/S", "/C"]` on Windows) to launch the application in the parameter of the instruction. Because of this, the primary process running in the container is not the application. Instead, it is the shell. This affects how the container exits, it affects how signals are processed, and it can really cause problems for images that do not include "/bin/sh". One use case where you might need to use the shell form is if you require shell-environment-variable substitution.

- There is also a use case for using both instructions in your Dockerfile. When you use both, you can define a specific application that gets executed when the container is run, and allow the user to easily provide the parameters that get used with the defined application. In this scenario, you would use the ENTRYPOINT instruction to set the application being executed and provide a default set of parameters for the application using the `CMD` instruction. With this configuration, the user of the container can benefit from the default parameters supplied in the `CMD` instruction, or they can easily override those parameters used in the application by supplying them as arguments in the container run command. It is highly recommended that you use the exec form of both instructions when you use them together.

## The `HEALTHCHECK` instruction

- The `HEALTHCHECK` instruction, which is a fairly new addition to the Dockerfile, is used to define the command to run inside a container to test the container's application health. When a container has a `HEALTHCHECK`, it gets a special status variable. Initially, that variable will be set to starting. Any time a `HEALTHCHECK ` is performed successfully, the status will be set to healthy. When a `HEALTHCHECK` is performed and fails, the fail count value will be incremented and then checked against a retries value. If the fail count equals or exceeds the retries value, the status is set to unhealthy. The syntax of the `HEALTHCHECK` instruction is as follows

```
# HEALTHCHECK instruction syntax
HEALTHCHECK [OPTIONS] CMD command (check container health by running a command inside the container)
HEALTHCHECK NONE (disable any HEALTHCHECK inherited from the base image)

```
There are four options that can be used when setting the `HEALTHCHECK`, and these options are as follows:

```
# HEALTHCHECK CMD options
--interval=DURATION (default: 30s)
--timeout=DURATION (default: 30s)
--start-period=DURATION (default: 0s)
--retries=N (default: 3)

```

- The `--interval` option allows you to define the amount of time between the HEALTHCHECK tests. The `--timeout` option allows you to define the amount of time that is considered too long for a HEALTHCHECK test. If the timeout is exceeded, the test is automatically considered a failure. The `--start-period` option allows for the definition of a no-fail time period during the container startup. Finally, the `--retries` option allows you to define how many consecutive failures it takes to update the `HEALTHCHECK` status to unhealthy.

- The `CMD` part of the `HEALTHCHECK` instruction follows the same rules as the `CMD` instruction. Please review the preceding section regarding the `CMD` instruction for complete details. The `CMD` that is used will provide a status when it exits, which will be either a 0 for success or a 1 for fail. Here is a `Dockerfile` example that uses the `HEALTHCHECK` instruction

```
# HEALTHCHECK instruction Dockerfile for Docker Quick Start
FROM alpine
RUN apk add curl
EXPOSE 80/tcp
HEALTHCHECK --interval=30s --timeout=3s \
CMD curl -f http://localhost/ || exit 1
CMD while true; do echo 'DQS Expose Demo' | nc -l -p 80; done

```

Running a container from an image built with the preceding Dockerfile looks like this:

```
docker container run --rm -d -p 80:80 --name health sangam14-healthcheck

```

You can see that the `HEALTHCHECK` initially reported a status of starting, but once the `HEALTHCHECKCMD` reported success, the status updated to healthy.


## The `ONBUILD` instruction























