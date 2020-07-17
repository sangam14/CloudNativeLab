---
layout: default
title:   Securing Kubernetes Pods
parent: CKA / CKAD Certification Workshop Track
nav_order: 9
---
# Securing Kubernetes Pods

# Hardening container images

- Container image hardening means to follow security best practices or baselines to configure a container image in order to reduce the attack surface. Image scanning tools only focus on finding publicly disclosed issues in applications bundled inside the image. But, following the best practices along with secure configuration while building the image ensures that the application has a minimal attack surface.

- Before we start talking about the secure configuration baseline, let's look at what a container image is, as well as a Dockerfile, and how it is used to build an image.

- Before we start talking about the secure configuration baseline, let's look at what a container image is, as well as a Dockerfile, and how it is used to build an image.

# Container images and Dockerfiles

 -  FROM: Initialize a new build stage from the base image or parent image. Both mean the foundation or the file layer on which you're bundling your own image.
 -  RUN: Execute commands and commit the results on top of the previous file layer.
 -  ENV: Set environment variables for the running containers.
 -  CMD: Specify the default commands that the containers will run.
 -  COPY/ADD: Both commands copy files or directories from the local (or remote) URL to the filesystem of the image.
 -  EXPOSE: Specify the port that the microservice will be listening on during container runtime.
 -  ENTRYPOINT: Similar to CMD, the only difference is that ENTRYPOINT makes a container that will run as an executable.
 -  WORKDIR: Sets the working directory for the instructions that follow.
 -  USER: Sets the user and group ID for any CMD/ENTRYPOINT of containers.
 
 
 
From the preceding Dockerfile, we can tell that the image was built on top of ubuntu. Then, it ran a bunch of apt-get commands to install the dependencies, 
and created a directory called /var/www. Next, copy the app.js file from the current directory to /var/www/app.js in the filesystem of the image. Finally, 
configure the default command to run this Node.js application. I believe you will see how straightforward and powerful Dockerfile is when it comes to helping you build an image.
  
 ```
 FROM ubuntu
# install dependencies
RUN apt-get install -y software-properties-common python
RUN add-apt-repository ppa:chris-lea/node.js
RUN echo "deb http://us.archive.ubuntu.com/ubuntu/ precise universe" >> /etc/apt/sources.list
RUN apt-get update
RUN apt-get install -y nodejs
# make directory
RUN mkdir /var/www
# copy app.js
ADD app.js /var/www/app.js
# set the default command to run
CMD ["/usr/bin/node", "/var/www/app.js"]
 

 ```
# CIS Docker benchmarks
 
 Center for Internet Security (CIS) put together a guideline regarding Docker container administration and management.
 Now, let's take a look at the security recommendations from CIS Docker benchmarks regarding container images:
# Create a user for a container image to run a microservice:
 
 - It is good practice to run a container as non-root. Although user namespace mapping is available, 
 it is not enabled by default. Running as root means that if an attacker were to successfully escape from the container, they would gain root access to the host.
 Use the USER instruction to create a user in the Dockerfile.
 
# Use trusted base images to build your own image:
 
 Images downloaded from public repositories cannot be fully trusted. It is well known that images from public repositories may contain malware or crypto miners.
 Hence, it is recommended that you build your image from scratch or use minimal trusted images, such as Alpine. Also, perform the image scan after your image has been built.
 
# Do not install unnecessary packages in your image:
 Installing unnecessary packages will increase the attack surface. It is recommended that you keep your image slim. Occasionally, 
 you will probably need to install some tools during the process of building an image. Do remember to remove them at the end of the Dockerfile. 
 
# Scan and rebuild an image in order to apply security patches:
 
 It is highly likely that new vulnerabilities will be discovered in your base image or in the packages you install in your image. It is good practice to scan your image frequently. Once you identify any vulnerabilities, try to patch the security fixes by rebuilding the image. 
 Image scanning is a critical mechanism for identifying vulnerabilities at the build stage.
 
# Enable content trust for Docker:
 Content trust uses digital signatures to ensure data integrity between the client and the Docker registry. It ensures the provenance of the container image. However, it is not enabled by default.
 You can turn it on by setting the environment variable, `DOCKER_CONTENT_TRUST`, to 1.
 
# Add a HEALTHCHECK instruction to the container image:
 
 A HEALTHCHECK instruction defines a command to ask Docker Engine to check the health status of the container periodically. Based on the health status check result,
 Docker Engine then exits the non-healthy container and initiates a new one.
 
 # Remove setuid and setgid permission from files in the image:
 
 `setuid` and `setgid `permissions can be used for privilege escalation as files with such permissions are allowed to be executed with owners' privileges instead of launchers' privileges.
 You should carefully review the files with `setuid` and `setgid` permissions and remove those files that don't require such permissions.
 
# Use COPY instead of ADD in the Dockerfile:
 
 The `COPY `instruction can only copy files from the local machine to the filesystem of the image, while the `ADD` instruction can not only copy files from the local machine but also retrieve files from the remote URL to the filesystem of the image.
 Using ADD may introduce the risk of adding malicious files from the internet to the image.
 
# Do not store secrets in the Dockerfile: 
 
 There are many tools that are able to extract image file layers. If there are any secrets stored in the image, secrets are no longer secrets. Storing secrets in the Dockerfile renders containers potentially exploitable. 
 A common mistake is to use the ENV instruction to store secrets in environment variables. 
 
# Install verified packages only: 
 
 This is similar to using the trusted base image only. 
 Observe caution as regards the packages you are going to install within your image. Make sure they are from trusted package repositories. 
 
 
 
 
