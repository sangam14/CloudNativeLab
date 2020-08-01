---
layout: default
title: Jobs and CronJobs
parent: CKA / CKAD Certification Workshop Track
nav_order: 17
---

# Jobs

# 1.Create a Hello World Job

We will create a hello world job, which says „hello world“, wait for 5 seconds and then quit with a „bye“. For that,
we will create a YAML file with a Kubernetes object of kind batch/job:

```
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-world-job
spec:
  template:
    spec:
      containers:
      - name: hello-world-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "echo 'Hello World'; sleep 5; echo 'Bye'"
      restartPolicy: Never

```

# Create and watch the Job PODs :
Now let us create the job and observe the created PODs with a watch command:

```
kubectl apply -f job.yaml

# output: job.batch/hello-world-job created
watch 'kubectl get pod'

# output every some seconds:
NAME                    READY   STATUS              RESTARTS   AGE
hello-world-job-zrsg9   0/1     ContainerCreating   0          4s

NAME                    READY   STATUS              RESTARTS   AGE
hello-world-job-zrsg9   0/1     ContainerCreating   0          7s

NAME                    READY   STATUS    RESTARTS   AGE
hello-world-job-zrsg9   1/1     Running   0          9s

NAME                    READY   STATUS    RESTARTS   AGE
hello-world-job-zrsg9   1/1     Running   0          10s

NAME                    READY   STATUS    RESTARTS   AGE
hello-world-job-zrsg9   1/1     Running   0          11s

NAME                    READY   STATUS      RESTARTS   AGE
hello-world-job-zrsg9   0/1     Completed   0          13s

NAME                    READY   STATUS      RESTARTS   AGE
hello-world-job-zrsg9   0/1     Completed   0          14s

NAME                    READY   STATUS      RESTARTS   AGE
hello-world-job-zrsg9   0/1     Completed   0          15s


```

The watch command can be finished by pressing <Ctrl> – c.

# Get Job Status :
Let us view the number of completions of the job: the hello-world-job has completed one out or one runs:

```
kubectl get job
NAME              COMPLETIONS   DURATION   AGE
hello-world-job   1/1           13s        5m52s

```

We can see more information using a describe command. Among others, we will see, how many PODs are running, how many have succeeded and how many have failed: 
0 Running / 1 Succeeded / 0 Failed


```
kubectl describe jobs

# output:
Name:           hello-world-job
Namespace:      default
Selector:       controller-uid=efdd427d-b154-11e9-8bd4-0242ac110011
Labels:         controller-uid=efdd427d-b154-11e9-8bd4-0242ac110011
                job-name=hello-world-job
Annotations:    kubectl.kubernetes.io/last-applied-configuration:
                  {"apiVersion":"batch/v1","kind":"Job","metadata":{"annotations":{},"name":"hello-world-job","namespace":"default"},"spec":{"template":{"sp...
Parallelism:    1
Completions:    1
Start Time:     Sun, 28 Jul 2020 16:29:57 +0000
Completed At:   Sun, 28 Jul 2020 16:30:10 +0000
Duration:       13s
Pods Statuses:  0 Running / 1 Succeeded / 0 Failed
Pod Template:
  Labels:  controller-uid=efdd427d-b154-11e9-8bd4-0242ac110011
           job-name=hello-world-job
  Containers:
   hello-world-container:
    Image:      centos:7
    Port:       
    Host Port:  
    Command:
      bash
    Args:
      -c
      echo 'Hello World'; sleep 5; echo 'Bye'
    Environment:  
    Mounts:       
  Volumes:        
Events:
  Type    Reason            Age    From            Message
  ----    ------            ----   ----            -------
  Normal  SuccessfulCreate  8m17s  job-controller  Created pod: hello-world-job-zrsg9


```

# Get Job Output:
As expected, the POD has sent greetings to STDOUT:

```
kubectl logs hello-world-job-zrsg9

# output:
Hello World
Bye


```

Note: the POD name will  be different in your case.

# Review Job Parameters

```
kubectl get -o yaml job hello-world-job

# output:
apiVersion: batch/v1
kind: Job
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"batch/v1","kind":"Job","metadata":{"annotations":{},"name":"hello-world-job","namespace":"default"},"spec":{"template":{"spec":{"containers":[{"args":["-c","echo 'Hello World'; sleep 5; echo 'Bye'"],"command":["bash"],"image":"centos:7","name":"hello-world-container"}],"restartPolicy":"Never"}}}}
  creationTimestamp: "2019-07-28T16:29:57Z"
  labels:
    controller-uid: efdd427d-b154-11e9-8bd4-0242ac110011
    job-name: hello-world-job
  name: hello-world-job
  namespace: default
  resourceVersion: "5712"
  selfLink: /apis/batch/v1/namespaces/default/jobs/hello-world-job
  uid: efdd427d-b154-11e9-8bd4-0242ac110011
spec:
  backoffLimit: 6
  completions: 1
  parallelism: 1
  selector:
    matchLabels:
      controller-uid: efdd427d-b154-11e9-8bd4-0242ac110011
  template:
    metadata:
      creationTimestamp: null
      labels:
        controller-uid: efdd427d-b154-11e9-8bd4-0242ac110011
        job-name: hello-world-job
    spec:
      containers:
      - args:
        - -c
        - echo 'Hello World'; sleep 5; echo 'Bye'
        command:
        - bash
        image: centos:7
        imagePullPolicy: IfNotPresent
        name: hello-world-container
        resources: {}
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
      dnsPolicy: ClusterFirst
      restartPolicy: Never
      schedulerName: default-scheduler
      securityContext: {}
      terminationGracePeriodSeconds: 30
status:
  completionTime: "2020-07-28T16:30:10Z"
  conditions:
  - lastProbeTime: "2020-07-28T16:30:10Z"
    lastTransitionTime: "2020-07-28T16:30:10Z"
    status: "True"
    type: Complete
  startTime: "2020-07-28T16:29:57Z"
  succeeded: 1

```

# Increase the Number of Completions

```
piVersion: batch/v1
kind: Job
metadata:
  name: hello-world-job
spec:
  completions: 5 
  template:
    spec:
      containers:
      - name: hello-world-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "echo 'Hello World'; sleep 5; echo 'Bye'"
      restartPolicy: Never

```
Now let us try to apply the change:

```
kubectl apply -f job.yaml

# output:
The Job "hello-world-job" is invalid: spec.completions: Invalid value: 5: field is immutable


```

This did not work, because the field is immutable. Instead, we need to delete the job and create a new job,  based on the new YAML file:

```
kubectl delete job hello-world-job
# output: job.batch "hello-world-job" deleted

kubectl apply -f job.yaml
# output: job.batch/hello-world-job created


```
Now, we can see that the job has been performed 5 times:

```
kubectl get jobs

# output:
NAME              COMPLETIONS   DURATION   AGE
hello-world-job   5/5           36s        103s


```

Each job has been performed in its own POD:

```
kubectl get pods

# output:
NAME                    READY   STATUS      RESTARTS   AGE
hello-world-job-4n25s   0/1     Completed   0          2m20s
hello-world-job-6lkmx   0/1     Completed   0          2m41s
hello-world-job-jznwv   0/1     Completed   0          2m48s
hello-world-job-sqghk   0/1     Completed   0          2m27s
hello-world-job-tl6nl   0/1     Completed   0          2m34s


```

# Parallel Execution of Jobs

From the values in the AGE column, we can deduce that the PODs have run sequentially. Now, let us try to run four of the five PODs in parallel.
For that, we add the blue line:

```

apiVersion: batch/v1
kind: Job
metadata:
  name: hello-world-job
spec:
  completions: 5
  parallelism: 4
  template:
    spec:
      containers:
      - name: hello-world-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "echo 'Hello World'; sleep 5; echo 'Bye'"
      restartPolicy: Never


```
We apply the configuration by deleting and re-creating the job:

```
kubectl delete job hello-world-job
# output:job.batch "hello-world-job" deleted

kubectl apply -f job.yaml
# output: job.batch/hello-world-job created

kubectl get pods
# output:
NAME                    READY   STATUS      RESTARTS   AGE
hello-world-job-6zbtz   0/1     Completed   0          9s
hello-world-job-b7vwd   0/1     Completed   0          9s
hello-world-job-rphkn   1/1     Running     0          3s  <----- late comer
hello-world-job-sd7z5   0/1     Completed   0          9s
hello-world-job-w4k6j   0/1     Completed   0          9s

kubectl get pods
# output:
NAME                    READY   STATUS      RESTARTS   AGE
hello-world-job-6zbtz   0/1     Completed   0          18s
hello-world-job-b7vwd   0/1     Completed   0          18s
hello-world-job-rphkn   0/1     Completed   0          12s  <----- late comer
hello-world-job-sd7z5   0/1     Completed   0          18s
hello-world-job-w4k6j   0/1     Completed   0          18s

```
Indeed, we see above that four of the five PODs were executed in parallel, while the fifth POD has waited for a free slot.

# Configure an Execution Timeout

Now, let us consider a situation, where a job takes too long to make any sense. For such situations, we can define an
execution timeout called `activeDeadlineSeconds`:

```
# job.yaml
apiVersion: batch/v1
kind: Job
metadata:
  name: hello-world-job
spec:
  completions: 5
  parallelism: 2
  activeDeadlineSeconds: 15
  template:
    spec:
      containers:
      - name: hello-world-container
        image: centos:7
        command: ["bash"]
        args:
        - "-c"
        - "echo 'Hello World'; sleep 5; echo 'Bye'"
      restartPolicy: Never




```
We set the number of completions to five with may two parallel PODs, while we allow for the execution time of 15 seconds.

```
kubectl get pods

# output:
NAME                    READY   STATUS        RESTARTS   AGE
hello-world-job-2rktd   0/1     Terminating   0          17s
hello-world-job-5hhbf   0/1     Completed     0          29s
hello-world-job-7qnj9   0/1     Completed     0          23s
hello-world-job-9d555   0/1     Completed     0          29s
hello-world-job-k4j7t   0/1     Completed     0          23s

```
After some time, the Terminating POD just disappears:

```
kubectl get pods

# output:
NAME                    READY   STATUS      RESTARTS   AGE
hello-world-job-5hhbf   0/1     Completed   0          34s
hello-world-job-7qnj9   0/1     Completed   0          28s
hello-world-job-9d555   0/1     Completed   0          34s
hello-world-job-k4j7t   0/1     Completed   0          28s

```
Let us review the events globally:
```
kubectl get events

# output 
<output omitted>
7m59s       Normal    SuccessfulCreate   job/hello-world-job         Created pod: hello-world-job-9d555
7m59s       Normal    SuccessfulCreate   job/hello-world-job         Created pod: hello-world-job-5hhbf
7m53s       Normal    SuccessfulCreate   job/hello-world-job         Created pod: hello-world-job-k4j7t
7m53s       Normal    SuccessfulCreate   job/hello-world-job         Created pod: hello-world-job-7qnj9
7m47s       Normal    SuccessfulCreate   job/hello-world-job         Created pod: hello-world-job-2rktd
7m44s       Normal    SuccessfulDelete   job/hello-world-job         Deleted pod: hello-world-job-2rktd
7m44s       Warning   DeadlineExceeded   job/hello-world-job         Job was active longer than specified deadline

```
Similar information can be seen on the kubectl describe job command.


# 2.CronJobs

In this phase, we will perform changes to our job.yaml file, so the job will be executed periodically.
```
cp job.yaml cronjob.yaml

```

```
# cronjob.yaml
apiVersion: batch/v1beta1         # <---------- batch/v1 -> batch/v1beta1
kind: CronJob                     # <---------- Job -> CronJob
metadata:
  name: hello-world-cronjob       # <---------- new name
spec:
  schedule: "*/2 * * * *"         # <---------- UNIX style cron tab
  jobTemplate:                    # <---------- JobTemplate
    spec:                         # <---------- JobTemplate Spec
      template:                   # <---------- from here:
                                  # <---------- same as Job spec, but indented for 4 spaces
        spec:
          containers:
          - name: hello-world-container
            image: centos:7
            command: ["bash"]
            args:
            - "-c"
            - "echo 'Hello World'; sleep 5; echo 'Bye'"
          restartPolicy: Never

```
We can see that a CronJob has a UNIX-style crontab string and a JobTemplate, which is a template to create jobs from it. Therefore, it is not surprising to 
find the job specs within the template again.

# Create a CronJob from a YAML File

Now, let us create the resource:
```
kubectl apply -f cronjob.yaml
# output: cronjob.batch/hello-world-cronjob created
```

# Check the Status

We check the resource:

```
kubectl get cronjobs

# output:
NAME                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello-world-cronjob   */2 * * * *   False     0        <none>          6s


```
It has not yet run. Therefore, no job and no POD has been created yet:

```
kubectl get jobs
# output: No resources found.

kubectl get pods
# output: No resources found.


```
Let us wait for 2 minutes and try again (several times):

```
kubectl get pods
# output:
NAME                                   READY   STATUS              RESTARTS   AGE
hello-world-cronjob-1564340520-jljqw   0/1     ContainerCreating   0          4s

kubectl get pods
# output:
NAME                                   READY   STATUS    RESTARTS   AGE
hello-world-cronjob-1564340520-jljqw   1/1     Running   0          9s

kubectl get pods
# output:
NAME                                   READY   STATUS    RESTARTS   AGE
hello-world-cronjob-1564340520-jljqw   1/1     Running   0          13s

kubectl get pods
# output:
NAME                                   READY   STATUS      RESTARTS   AGE
hello-world-cronjob-1564340520-jljqw   0/1     Completed   0          1m47s

```
Something similar can be seen on the Job level:

```
kubectl get jobs
# output:
NAME                             COMPLETIONS   DURATION   AGE
hello-world-cronjob-1564340640   1/1           7s         4m34s
hello-world-cronjob-1564340760   1/1           7s         2m34s
hello-world-cronjob-1564340880   1/1           7s         34s


```
Now let us check the status of the cronjob again:

```
kubectl get cronjobs

# output:
NAME                  SCHEDULE      SUSPEND   ACTIVE   LAST SCHEDULE   AGE
hello-world-cronjob   */2 * * * *   False     0        19s             10m

```
We can see that the last job was scheduled 19 sec ago.

As a summary, a CronJob is an object that creates Jobs periodically by a schedule defined in the UNIX-style crontab.
