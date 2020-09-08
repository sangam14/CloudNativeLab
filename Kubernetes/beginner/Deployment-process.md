# sequential breakdown of the Kubernetes Deployment process.

efore we move onto Deployment updates, we’ll go through our usual ritual of seeing the process through a 
sequence diagram. We won’t repeat the explanation of the events that happened after the ReplicaSet object was created as those steps [here](https://containerlabs.kubedaily.com/Kubernetes/beginner/Sequential-Breakdown-of-the-Process.html).

- Kubernetes client ( kubectl ) sent a request to the API server requesting the creation of a Deployment defined in the deploy.yml file.
- The deployment controller is watching the API server for new events, and it detected that there is a new Deployment object.
- The deployment controller creates a new ReplicaSet object.

![](https://github.com/sangam14/ContainerLabs/blob/master/img/Deployement-Sequence.png)

The above illustration is self-explanatory with the sequence of the processes linked to the deployment process.


