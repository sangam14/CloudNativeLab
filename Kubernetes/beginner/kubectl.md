

# Understanding kubectl 
Kubernetes’ command-line tool, kubectl , is used to manage a cluster and applications running inside it. We’ll use kubectl a lot throughout the labs , 
so we won’t go into details just yet. Instead, we’ll discuss its commands through examples that will 
follow shortly. For now, think of it as your interlocutor with a Kubernetes cluster.

# Installation 
Let’s install kubectl .
Feel free to skip the installation steps if you already have kubectl . Just make
sure that it is version 1.8 or above.

## MacOS 

If you are a MacOS user, please execute the commands that follow

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/darwin/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

```

If you already have Homebrew package manager installed, you can “brew” it with the command that follows.

```
brew install kubectl

```
## Linux 

If, on the other hand, you’re a Linux user, the commands that will install kubectl are as follows.

```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

sudo mv ./kubectl /usr/local/bin/kubectl

```
## Windows 
Finally, Windows users should download the binary through the command that follows.
```
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/windows/amd64/kubectl

```

Feel free to copy the binary to any directory. The important thing is to add it to your PATH .

## Verification 
Let’s check kubectl version and, at the same time, validate that it is working correctly. No matter which OS you’re using, the command is as follows.

```
kubectl version

```
fortunately, kubectl can use a few different formats for its output. For example, we can tell it to output the command in yaml format

```
kubectl version --output=yaml

```

