---
layout: default
title: Configuring the security attributes of pods
parent: CKA / CKAD Certification Workshop Track
nav_order: 11
---

## Configuring the security attributes of pods

- application developers should be aware of what privileges a microservice must have in order to perform tasks.
  Ideally, application developers and security engineers work together to harden the microservice at the pod and 
  container level by configuring the security context provided by Kubernetes. 
  
  
- We classify the major security attributes into four categories:

    - Setting host namespaces for pods
    - Security context at the container level
    - Security context at the pod level
    - AppArmor profile
    
By employing such a means of classification, you will find them easy to manage. 

## Setting host-level namespaces for pods
The following attributes in the pod specification are used to configure the use of host namespaces:
  - hostPID: By default, this is false. Setting it to true allows the pod to have visibility on all the processes in the worker node.
  - hostNetwork: By default, this is false. Setting it to true allows the pod to have visibility on all the network stacks in the worker node.
  - hostIPC: By default, this is false. Setting it to true allows the pod to have visibility on all the IPC resources in the worker node. 
  
  
## The following is an example of how to configure the use of host namespaces at the pod level in an ubuntu-1 pod YAML file:

```
apiVersion: v1
kind: Pod
metadata:
  name: ubuntu-1
  labels:
    app: util
spec:
  containers:
  - name: ubuntu
    image: ubuntu
    imagePullPolicy: Always
  hostPID: true
  hostNetwork: true
  hostIPC: true

```

The preceding workload YAML configured the ubuntu-1 pod to use a host-level PID namespace, network namespace, and IPC namespace.
Keep in mind that you shouldn't set these attributes to true unless necessary. Setting these attributes to true also disarms the security boundaries of other workloads
in the same worker node.

## Security context for containers

- Multiple containers can be grouped together inside the same pod. Each container can have its own security context, which defines privileges and access controls. 
The design of a security context at a container level provides a more fine-grained security control for Kubernetes workloads. For example, you may have three 
containers running inside the same pod and one of them has to run in privileged mode, while the others run in non-privileged mode. This can be done by configuring 
a security context for individual containers.

The following are the principal attributes of a security context for containers:

- privileged: By default, this is false. Setting it to true essentially makes the processes inside the container equivalent to the root user on the worker node. 

- capabilities: There is a default set of capabilities granted to the container by the container runtime. The default capabilities granted are as follows: CAP_SETPCAP, CAP_MKNOD, CAP_AUDIT_WRITE, CAP_CHOWN, CAP_NET_RAW, CAP_DAC_OVERRIDE, CAP_FOWNER, CAP_FSETID, CAP_KILL, CAP_SETGID, CAP_SETUID,
CAP_NET_BIND_SERVICE, CAP_SYS_CHROOT, and CAP_SETFCAP. You may add extra capabilities or drop some of the defaults by configuring this attribute. Capabilities such as CAP_SYS_ADMIN and CAP_NETWORK_ADMIN should be added with caution. For the default capabilities, you should also drop those that are unnecessary.

- allowPrivilegeEscalation: By default, this is true. Setting it directly controls the no_new_privs flag, which will be set to the processes in the container. Basically, this attribute controls whether the process can gain more privileges than its parent process. Note that if the container runs in privileged mode, 
or has the CAP_SYS_ADMN capability added, this attribute will be set to true automatically. It is good practice to set it to false.

- readOnlyRootFilesystem: By default, this is false. Setting it to true makes the root filesystem of the container read-only, which means that the library files, configuration files, and so on are read-only and cannot be tampered with. It is a good security practice to set it to true

- runAsNonRoot: By default, this is false. Setting it to true enables validation that the processes in the container cannot run as a root user (UID=0). Validation is done by kubelet. With runAsNonRoot set to true, kubelet will prevent the container from starting if run as a root user. It is a good security practice to set it to true. This attribute is also available in PodSecurityContext, which takes effect at pod level.
If this attribute is set in both SecurityContext and PodSecurityContext, the value specified at the container level takes precedence.

- runAsUser: This is designed to specify to the UID to run the entrypoint process of the container image. The default setting is the user specified in the image's metadata (for example, the USER instruction in the Dockerfile). This attribute is also available in PodSecurityContext, which takes effect at the pod level. If this attribute is set in both SecurityContext and PodSecurityContext, 
the value specified at the container level takes precedence.

- seLinuxOptions: This is designed to specify the SELinux context to the container. By default, the container runtime will assign a random SELinux context to the container if not specified. This attribute is also available in PodSecurityContex, which takes effect at the pod level. If this attribute is set in both SecurityContext and PodSecurityContext, the value specified at the container level takes precedence.

Since you now understand what these security attributes are, you may come up with your own hardening strategy aligned with your business requirements. In general, the security best practices are as follows:

- Do not run in privileged mode unless necessary.
- Do not add extra capabilities unless necessary.
- Drop unused default capabilities.
- Run containers as a non-root user.
- Enable a runAsNonRoot check
- Set the container root filesystem as read-only.


## Now, let's take a look at an example of configuring SecurityContext for containers:

```
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
  labels:
    app: web
spec:
  hostNetwork: false
  hostIPC: false
  hostPID: false
  containers:
  - name: nginx
    image: kaizheh/nginx 
    securityContext:
      privileged: false
      capabilities:
        add:
        - NETWORK_ADMIN
      readOnlyRootFilesystem: true 
      runAsUser: 100
      runAsGroup: 1000


```

The nginx container inside nginx-pod runs as a user with a UID of 100 and a GID of 1000. In addition to this, the nginx container gains extra NETWORK_ADMIN capability and the root filesystem is set to read-only. The YAML file here only shows an example of how to configure the security context. 
Note that adding NETWORK_ADMIN is not recommended for containers running in production environments.

## Security context for pods

A security context is used at the pod level, which means that security attributes will be applied to all the containers inside the pod.

The following is a list of the principal security attributes at the pod level:

- fsGroup: This is a special supplemental group applied to all containers. The effectiveness of this attribute depends on the volume type. Essentially, it allows kubelet to set the ownership of the mounted volume to the pod with the supplemental GID.

- ysctls: sysctls is used to configure kernel parameters at runtime. In such a context, sysctls and kernel parameters are used interchangeably. These sysctls commands are namespaced kernel parameters that apply to the pod. The following sysctls commands are known to be namespaced: kernel.shm*, kernel.msg*, kernel.sem, and kernel.mqueue.*. Unsafe sysctls are disabled by default and should not be enabled in production environments. 

- runAsUser: This is designed to specify the UID to run the entrypoint process of the container image. The default setting is the user specified in the image's metadata (for example, the USER instruction in the Dockerfile). This attribute is also available in SecurityContext, which takes effect at the container level. If this attribute is set in both SecurityContext and PodSecurityContext, the value specified at the container level takes precedence.

- runAsGroup: Similar to runAsUser, this is designed to specify the GID to run the entrypoint process of the container. This attribute is also available in SecurityContext, which takes effect at the container level. If this attribute is set in both SecurityContext and PodSecurityContext, the value specified at the container level takes precedence.

- runAsNonRoot: Set to false by default, setting it to true enables validation that the processes in the container cannot run as a root user (UID=0). Validation is done by kubelet. By setting it to true, kubelet will prevent the container from starting if run as a root user. It is a good security practice to set it to true. This attribute is also available in SecurityContext, which takes effect at the container level. If this attribute is set in both SecurityContext and PodSecurityContext, the value specified at the container level takes precedence.

- seLinuxOptions: This is designed to specify the SELinux context to the container. By default, the container runtime will assign a random SELinux context to the container if not specified. This attribute is also available in SecurityContext, which takes effect at the container level. If this attribute is set in both SecurityContext and PodSecurityContext, the value specified at the container level takes precedence.

Notice that the attributes runAsUser, runAsGroup, runAsNonRoot, and seLinuxOptions are available both in SecurityContext at the container level and PodSecurityContext at the pod level. This gives users both the flexibility and extreme importance of security control. fsGroup and sysctls are not as commonly used as the others, so only use them when you have to. 

