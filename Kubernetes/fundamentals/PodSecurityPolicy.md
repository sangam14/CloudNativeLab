---
layout: default
title:  Pod Security Policy
parent: CKA / CKAD Certification Workshop Track
nav_order: 10
---

## Pod Security Policy

- A Kubernetes PodSecurityPolicy is a cluster-level resource that controls security-sensitive aspects of the pod specification through which the access privileges of a Kubernetes pod are limited. As a DevOps engineer, you may want to use a PodSecurityPolicy to restrict most of the workloads run in limited access privileges, while only allowing a few workloads to be run with extra privileges. 

we will first take a closer look at a PodSecurityPolicy, and then we will introduce an open source tool, kube-psp-advisor, which can help build an adaptive PodSecurityPolicy for the running Kubernetes cluster. 

## Understanding Pod Security Policy

- You can think of a PodSecurityPolicy as a policy to evaluate the security attributes defined in the pod's specification. Only those pods whose security attributes meet the requirements of PodSecurityPolicy will be admitted to the cluster. For example, PodSecurityPolicy can be used to block the launch of most privileged pods, while only allowing those necessary or limited pods access to the host filesystem.

The following are the principal security attributes that are controlled by PodSecurityPolicy:
  

   - privileged: Determines whether a pod can run in privileged mode.
   - hostPID: Determines whether a pod can use a host PID namespace.
   - hostNetwork: Determines whether a pod can use a host network namespace.
   - hostIPC: Determines whether a pod can use a host IPC namespace. The default setting is true.
   - allowedCapabilities: Specifies a list of capabilities that could be added to containers. The default setting is empty.
   - defaultAddCapabilities: Specifies a list of capabilities that will be added to containers by default. The default setting is empty.
   - requiredDropCapabilities: Specifies a list of capabilities that will be dropped from containers. Note that a capability cannot be specified in both the allowedCapabilities and requiredDropCapabilities fields. The default setting is empty.
   - readOnlyRootFilesystem: When set to true, the PodSecurityPolicy will force containers to run with a read-only root filesystem. If the attribute is set to false explicitly in the security context of the container, the pod will be denied from running. The default setting is false.
   - runAsUser: Specifies the allowable user IDs that may be set in the security context of pods and containers. The default setting allows all.
   - runAsGroup: Specifies the allowable group IDs that may be set in the security context of pods and containers. The default setting allows all.
   - allowPrivilegeEscalation: Determines whether a pod can submit a request to allow privilege escalation. The default setting is true.
   - allowedHostPaths: Specifies a list of host paths that could be mounted by the pod. The default setting allows all.
   - volumes: Specifies a list of volume types that can be mounted by the pod. For example, secret, configmap, and hostpath are the valid volume types. The default setting allows all.
   - seLinux: Specifies the allowable seLinux labels that may be set in the security context of pods and containers. The default setting allows all.
   - allowedUnsafeSysctl: Allows unsafe sysctls to run. The default setting allows none.
   
Now, let's take a look at an example of a PodSecurityPolicy:

```
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
    name: example
spec:
  allowedCapabilities:
  - NET_ADMIN
  - IPC_LOCK
  allowedHostPaths:
  - pathPrefix: /dev
  - pathPrefix: /run
  - pathPrefix: /
  fsGroup:
    rule: RunAsAny
  hostNetwork: true
  privileged: true
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - hostPath
  - secret

```

This PodSecurityPolicy allows the NET_ADMIN and IPC_LOCK capabilities, mounts /, /dev, and /run from the host and Kubernetes' secret volumes. It doesn't enforce any filesystem group ID or 
supplemental groups and it also allows the container to run as any user, access the host network namespace, and run as a privileged container. No SELinux policy is enforced in the policy.

To enable this Pod Security Policy, you can run the following command:

```
$ kubectl apply -f example-psp.yaml

```

Now, let's verify that the Pod Security Policy has been created successfully:

```
$ kubectl get psp

```

The output will appear as follows:

```
NAME      PRIV     CAPS                           SELINUX    RUNASUSER   FSGROUP    SUPGROUP   READONLYROOTFS   VOLUMES
example   true     NET_ADMIN, IPC_LOCK            RunAsAny   RunAsAny    RunAsAny   RunAsAny   false            hostPath,secret

```
After you have created the Pod Security Policy, there is one more step required in order to enforce it. You will have to grant the privilege of using the PodSecurityPolicy object to the users, groups, or service accounts. By doing so, the pod security policies are entitled to evaluate the workloads based on the associated service account. 
Here is an example of how to enforce a PodSecurityPolicy. 
First, you will need to create a cluster role that uses the PodSecurityPolicy:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: use-example-psp
rules:
- apiGroups: ['policy']
  resources: ['podsecuritypolicies']
  verbs:     ['use']
  resourceNames:
  - example

```
Then, create a RoleBinding or ClusterRoleBinding object to associate the preceding ClusterRole object created with the service accounts, users, or groups:

```

apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: use-example-psp-binding
roleRef:
  kind: ClusterRole
  name: use-example-psp
  apiGroup: rbac.authorization.k8s.io
subjects:
# Authorize specific service accounts:
- kind: ServiceAccount
  name: test-sa
  namespace: psp-test


```
- The preceding use-example-pspbinding.yaml file created a RoleBinding object to associate the use-example-psp cluster role with the test-sa service account in the psp-test namespace. With all of these set up, any workloads in the psp-test namespace whose service account is test-sa will run through the PodSecurityPolicy example's evaluation. 
And only those that meet the requirements will be admitted to the cluster.

- From the preceding example, think of there being different types of workloads running in your Kubernetes cluster, and each of them may require different privileges to access different types of resources. 
It would be a challenge to create and manage pod security policies for different workloads. Now, let's take a look at kube-psp-advisor and see how it can help create pod security policies for you.


## Kubernetes PodSecurityPolicy Advisor

- Kubernetes PodSecurityPolicy Advisor (also known as kube-psp-advisor) is an open source tool from Sysdig. It scans the security attributes of running workloads in the cluster and then, on this basis, recommends pod security policies for your cluster or workloads.

- First, let's install kube-psp-advisor as a kubectl plugin. If you haven't installed krew, a kubectl plugin management tool, please follow the instructions (https://github.com/kubernetes-sigs/krew#installation) in order to install it. Then, install kube-psp-advisor with krew as follows:

```

$ kubectl krew install advise-psp

```
Then, you should be able to run the following command to verify the installation:

```
$ kubectl advise-psp
A way to generate K8s PodSecurityPolicy objects from a live K8s environment or individual K8s objects containing pod specifications
Usage:
  kube-psp-advisor [command]
Available Commands:
  convert     Generate a PodSecurityPolicy from a single K8s Yaml file
  help        Help about any command
  inspect     Inspect a live K8s Environment to generate a PodSecurityPolicy
Flags:
  -h, --help           help for kube-psp-advisor
      --level string   Log level (default "info")

```
To generate pod security policies for workloads in a namespace, you can run the following command:

```
$ kubectl advise-psp inspect --grant --namespace psp-test

```

- The preceding command generates pod security policies for workloads running inside the psp-test namespace. If the workload uses a default service account, no PodSecurityPolicy will be generated for it. This is because the default service account will be assigned to the workload that does not have a dedicated service account associated with it. And you certainly don't want to have a default service account that is able to use a PodSecurityPolicy for privileged workloads.

- Here is an example of output generated by kube-psp-advisor for workloads in the psp-test namespace, including Role, RoleBinding, and PodSecurityPolicy in a single YAML file with multiple pod security policies. Let's take a look at one of the recommended PodSecurityPolicy:


```

# Pod security policies will be created for service account 'sa-1' in namespace 'psp-test' with following workloads:
#	Kind: ReplicaSet, Name: busy-rs, Image: busybox
#	Kind: Pod, Name: busy-pod, Image: busybox
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  creationTimestamp: null
  name: psp-for-psp-test-sa-1
spec:
  allowedCapabilities:
  - SYS_ADMIN
  allowedHostPaths:
  - pathPrefix: /usr/bin
    readOnly: true
  fsGroup:
    rule: RunAsAny
  hostIPC: true
  hostNetwork: true
  hostPID: true
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - configMap
  - secret
  - hostPath


```
Following is the Role generated by kube-psp-advisor:

```

apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  creationTimestamp: null
  name: use-psp-by-psp-test:sa-1
  namespace: psp-test
rules:
- apiGroups:
  - policy
  resourceNames:
  - psp-for-psp-test-sa-1
  resources:
  - podsecuritypolicies
  verbs:
  - use
---


```
Following is the RoleBinding generated by kube-psp-advisor:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  creationTimestamp: null
  name: use-psp-by-psp-test:sa-1-binding
  namespace: psp-test
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: Role
  name: use-psp-by-psp-test:sa-1
subjects:
- kind: ServiceAccount
  name: sa-1
  namespace: psp-test
---


```
The preceding section is the recommended PodSecurityPolicy, psp-for-psp-test-sa-1, for the busy-rs and busy-pod workloads, since these two workloads share the same service account, sa-1. Hence, Role and RoleBinding are created to use the Pod Security Policy, psp-for-psp-test-sa-1, respectively. The PodSecurityPolicy is generated based on the aggregation of the security attributes of workloads using the sa-1 service account:
```

---
# Pod security policies will NOT be created for service account 'default' in namespace 'psp-test' with following workdloads:
#	Kind: ReplicationController, Name: busy-rc, Image: busybox
---

```
- The preceding section mentions that the busy-rc workload uses a default service account, so there is no Pod Security Policy created for it. This is a reminder that if you want to generate pod security policies for workloads, don't use the default service account.

- Building a Kubernetes PodSecurityPolicy is not straightforward, although it would be ideal if a single restricted PodSecurityPolicy was to apply to the entire cluster and all workloads complied with it. DevOps engineers need to be creative in order to build restricted pod security policies while not breaking workloads' functionalities. kube-psp-advisor makes the implementation of Kubernetes pod security policies simple, adapts to your application requirements and, specifically, is fine-grained for each one to allow only the privilege of least access.
