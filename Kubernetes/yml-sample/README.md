
# The Ultimate Yaml for Kubernetes 

# Pod 

simple.yaml
```
---
apiVersion: v1
kind: Pod
metadata:
  name: pods-simple-pod
spec:
  containers:
    - command:
        - sleep
        - "3600"
      image: busybox
      name: pods-simple-container
     
```
multi-container.yaml
 
```
 ---
apiVersion: v1
kind: Pod
metadata:
  name: pods-multi-container-pod
spec:
  containers:
    - image: busybox
      command:
        - sleep
        - "3600"
      name: pods-multi-container-container-1
    - image: busybox
      command:
        - sleep
        - "3601"
      name: pods-multi-container-container-2
      
      
   ```   
imagepullsecret.yaml

```
# https://kubernetes.io/docs/tasks/configure-pod-container/pull-image-private-registry/
---
apiVersion: v1
kind: Pod
metadata:
  name: pods-imagepullsecret-pod
spec:
  containers:
    - command:
        - sleep
        - "3600"
      image: busybox
      name: pods-simple-container
  imagepullsecrets:
    - name: regcred  # does not exist, create with instructions above


```
host-aliases.yaml
```
---
# https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/#adding-additional-entries-with-hostaliases
apiVersion: v1
kind: Pod
metadata:
  name: pods-host-aliases-pod
spec:
  restartPolicy: Never
  hostAliases:
    - ip: "127.0.0.1"
      hostnames:
        - "foo.local"
        - "bar.local"
    - ip: "10.1.2.3"
      hostnames:
        - "foo.remote"
        - "bar.remote"
  containers:
    - name: cat-hosts
      image: busybox
      command:
        - cat
      args:
        - "/etc/hosts"
        
  ```   
  
# pod security policy

privileged.yaml
     
```
 ---
# This is the least restrictive policy you can create, equivalent to not using
# the pod security policy admission controller
# https://kubernetes.io/docs/concepts/policy/pod-security-policy/#example-policies
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: pod-security-policy-privileged-psp
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: '*'
spec:
  privileged: true
  allowPrivilegeEscalation: true
  allowedCapabilities:
    - '*'
  volumes:
    - '*'
  hostNetwork: true
  hostPorts:
    - min: 0
      max: 65535
  hostIPC: true
  hostPID: true
  runAsUser:
    rule: 'RunAsAny'
  seLinux:
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'RunAsAny'
  fsGroup:
    rule: 'RunAsAny'
 
 
 ```
 
 restricted.yaml
 
 ```
 ---
# This is an example of a restrictive policy that requires users to run as an
# unprivileged user, blocks possible escalations to root, and requires use of
# several security mechanisms.
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: pod-security-policy-restricted-psp
  annotations:
    seccomp.security.alpha.kubernetes.io/allowedProfileNames: 'docker/default,runtime/default'
    apparmor.security.beta.kubernetes.io/allowedProfileNames: 'runtime/default'
    seccomp.security.alpha.kubernetes.io/defaultProfileName: 'runtime/default'
    apparmor.security.beta.kubernetes.io/defaultProfileName: 'runtime/default'
spec:
  allowedHostPaths:
    # This allows "/foo", "/foo/", "/foo/bar" etc., but
    # disallows "/fool", "/etc/foo" etc.
    # "/foo/../" is never valid.
    - pathPrefix: "/foo"
      readOnly: true  # only allow read-only mounts
  allowPrivilegeEscalation: false
  # This is redundant with non-root + disallow privilege escalation,
  # but we can provide it for defense in depth.
  fsGroup:
    rule: 'MustRunAs'
    ranges:
      # Forbid adding the root group.
      - min: 1
        max: 65535
  hostIPC: false
  hostNetwork: false
  hostPID: false
  privileged: false
  readOnlyRootFilesystem: false
  # Required to prevent escalations to root.
  requiredDropCapabilities:
    - ALL
  runAsUser:
    # Require the container to run without root privileges.
    rule: 'MustRunAsNonRoot'
  seLinux:
    # This policy assumes the nodes are using AppArmor rather than SELinux.
    rule: 'RunAsAny'
  supplementalGroups:
    rule: 'MustRunAs'
    ranges:
      # Forbid adding the root group.
      - min: 1
        max: 65535
  # Allow core volume types.
  volumes:
    - 'configMap'
    - 'emptyDir'
    - 'projected'
    - 'secret'
    - 'downwardAPI'
    # Assume that persistentVolumes set up by the cluster admin are safe to use.
    - 'persistentVolumeClaim'
 
 
 ```
 
 psp.yaml
 
 ```
 ---
apiVersion: v1
kind: Namespace
metadata:
  name: pod-security-policy-psp-namespace
---
apiVersion: policy/v1beta1
kind: PodSecurityPolicy
metadata:
  name: pod-security-policy-psp
spec:
  privileged: false  # Don't allow privileged pods!
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  runAsUser:
    rule: RunAsAny
  fsGroup:
    rule: RunAsAny
  volumes:
    - '*'
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pod-security-policy-user
  namespace: pod-security-policy-psp-namespace
---
apiVersion: v1
items:
  - apiVersion: rbac.authorization.k8s.io/v1
    kind: RoleBinding
    metadata:
      name: pod-security-policy-psp-user-editor
      namespace: pod-security-policy-psp-namespace
    roleRef:
      apiGroup: rbac.authorization.k8s.io
      kind: ClusterRole
      name: edit
    subjects:
      - kind: ServiceAccount
        name: pod-security-policy-psp-namespace
        namespace: pod-security-policy-psp-namespace
kind: List
metadata:
  resourceVersion: ""
  selfLink: ""
---
apiVersion: v1
kind: Pod
metadata:
  name: pause
  namespace: pod-security-policy-psp-namespace-unprivileged
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
---
apiVersion: v1
kind: Pod
metadata:
  name: pause
  namespace: pod-security-policy-psp-namespace-privileged
spec:
  containers:
    - name: pause
      image: k8s.gcr.io/pause
      securityContext:
        privileged: true
  ``` 
  
 # deployments
 
 simple-deployment.yaml
 
 ```
 ---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: deployments-simple-deployment-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: deployments-simple-deployment-app
  template:
    metadata:
      labels:
        app: deployments-simple-deployment-app
    spec:
      containers:
        - name: busybox
          image: busybox
          command:
            - sleep
            - "3600"
 
 
 
 ```
 
 # namespace
 
 namespace.yaml
 ```
 ---
apiVersion: v1
kind: Namespace
metadata:
  name: namespace-namespace

```

memory-request-limit.yaml

```
---
apiVersion: v1
kind: Pod
metadata:
  name: -memory-request-limit-pod
spec:
  containers:
    - args: ["--vm", "1", "--vm-bytes", "150M", "--vm-hang", "1"]
      command: ["stress"]
      image: polinux/stress
      name: memory-request-limit-container
      resources:
        limits:
          memory: "200Mi"
        requests:
          memory: "100Mi"


```
# resource quotas

```

---
apiVersion: v1
kind: List
items:
  # https://kubernetes.io/docs/concepts/policy/resource-quotas/
  - apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: resource-quotas-quotas-pods-high
    spec:
      hard:
        cpu: "1000"
        memory: 200Gi
        pods: "10"
      scopeSelector:
        matchExpressions:
          - operator: In
            scopeName: PriorityClass
            values: ["high"]
  - apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: resource-quotas-quotas-pods-medium
    spec:
      hard:
        cpu: "10"
        memory: 20Gi
        pods: "10"
      scopeSelector:
        matchExpressions:
          - operator: In
            scopeName: PriorityClass
            values: ["medium"]
  - apiVersion: v1
    kind: ResourceQuota
    metadata:
      name: resource-quotas-quotas-pods-low
    spec:
      hard:
        cpu: "5"
        memory: 10Gi
        pods: "10"
      scopeSelector:
        matchExpressions:
          - operator: In
            scopeName: PriorityClass
            values: ["low"]

```

# daemon set

simple-daemon-set.yaml

```
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd-elasticsearch
spec:
  selector:
    matchLabels:
      name: fluentd-elasticsearch
  template:
    metadata:
      labels:
        name: fluentd-elasticsearch
    spec:
      tolerations:
        # this toleration is to have the daemonset runnable on master nodes
        # remove it if your masters can't run pods
        - effect: NoSchedule
          key: node-role.kubernetes.io/master
      containers:
        - image: quay.io/fluentd_elasticsearch/fluentd:v2.5.2
          name: fluentd-elasticsearch
      terminationGracePeriodSeconds: 30

```

# init-container

init-container.yaml

```
---
apiVersion: v1
kind: Pod
metadata:
  name: init-container-pod
spec:
  containers:
    - name: init-container-container
      image: busybox
      command: ['sh', '-c', 'echo The app is running! && sleep 3600']
  initContainers:
    - name: init-container-init-container
      image: busybox
      command: ['sh', '-c', "until nslookup pods-init-container-service.$(cat /var/run/secrets/kubernetes.io/serviceaccount/namespace).svc.cluster.local; do echo waiting for myservice; sleep 2; done"]
---
apiVersion: v1
kind: Service
metadata:
  name: init-container-service
spec:
  ports:
    - protocol: TCP
      port: 80
      targetPort: 12345


```

init-container-msg.yaml
```
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: init-container-msg-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: init-container-msg-app
  template:
    metadata:
      labels:
        app: init-container-msg-app
    spec:
      initContainers:
        - command:
            - "/bin/bash"
            - "-c"
            - "echo 'message from init' > /init-container-msg-mount-path/this"
          image: busybox
          name: init-container-msg-container-init
          volumeMounts:
            - mountPath: /init-container-msg-mount-path
              name: init-container-msg-volume
      containers:
        - command:
            - "/bin/sh"
            - "-c"
            - "while true; do cat /init-container-msg-mount-path/this; sleep 5; done"
          image: busybox
          name: init-container-msg-container-main
          volumeMounts:
            - mountPath: /init-container-msg-mount-path
              name: init-container-msg-volume
      volumes:
        - emptyDir: {}
          name: init-container-msg-volume



```
# lifecycle

```
---
apiVersion: v1
kind: Pod
metadata:
  name: lifecycle-pod
spec:
  containers:
    - image: nginx
      lifecycle:
        postStart:
          exec:
            command: ["/bin/sh", "-c", "echo Hello from the postStart handler > /usr/share/message"]
        preStop:
          exec:
            command: ["/bin/sh", "-c", "nginx -s quit; while killall -0 nginx; do sleep 1; done"]
      name: lifecycle-container


```






