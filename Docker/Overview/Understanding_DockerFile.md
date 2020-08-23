---
layout: default
title: Understanding DockerFile
parent: Docker Fundamental
nav_order: 3
---

# Describe the use of Dockerfile. 

- Remenber running single commands to run particular commands. but why dockerfile come into picture remember in we have docker hub something similiar to github ! 
in simple worlds dockerhub is the kind of library of dependancy . in technical worlds we all dependancies as base image. lets take one example you want to run simple java application you need some kind of dependacies like jdk7/8 installed on machine where you want to run java application . my be one of machine using different version of java. thats where base image come into pic . you can pull any kind of base image from dockerhub once and use multiple time anywhere. and thats all about base image. moving forword to understand more about dockerfile now  you have java as base image . and you can use that base image and top of that perform differnet operation.

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
