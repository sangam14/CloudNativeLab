---
layout: default
title: Installing Minikube
parent: Kubernetes For Beginner
nav_order: 6
---

# Installing Minikube


# Understanding Minikube 

Minikube supports several virtualization technologies. 
We’ll use VirtualBox throughout the book since it is the only virtualization supported in all operating systems. If you do not have it already, please head to the Download VirtualBox page and get the version that matches your OS 

### Note :- Please keep in mind that for VirtualBox or HyperV to work, virtualization must be enabled in the BIOS. Most laptops should have it enabled by default.

# Installation

Finally, we can install Minikube.

# MacOS 

If you’re using MacOS, please execute the command that follows.

```
brew install minikube

```
if installation fail check this [Link](https://osxdaily.com/2018/12/31/install-run-virtualbox-macos-install-kernel-fails/)

# Linux 

If, on the other hand, you prefer Linux, the command is as follows.

```
curl -Lo minikube https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/

```
# Windows 

Finally, you will not get a command if you are a Windows user. Instead, download the latest release from of the minikube-windows-amd64.exe file, rename it to minikube.exe , and add it to your path.

# Validation 
We’ll test whether Minikube works or not by checking its version.

```
minikube version

```
Now we’re ready to give the cluster a spin.


