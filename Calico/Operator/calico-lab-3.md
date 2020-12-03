Verify the Sample Application

Check the Deployment Status

To validate that the application has been deployed into your cluster, we will check the rollout status of each of the microservices.

Check the customer microservice:
```
kubectl rollout status -n yaobank deployment/customer
Example output:

deployment "customer" successfully rolled out
```
Check the summary microservice:
```

kubectl rollout status -n yaobank deployment/summary
Example output:

deployment "summary" successfully rolled out
```
Check the database microservice:
```
kubectl rollout status -n yaobank deployment/database
Example output:

deployment "database" successfully rolled out
```
Access the Sample Application Web GUI

Now we can browse to the service using the service’s NodePort. The NodePort exists on every node in the cluster. We’ll use the control node, but you get the exact same behavior connecting to any other node in the cluster.
```
curl 198.19.0.1:30180
The resulting output should contain the following balance information:

  <body>
        <h1>Welcome to YAO Bank</h1>
        <h2>Name: Spike Curtis</h2>
        <h2>Balance: 2389.45</h2>
        <p><a href="/logout">Log Out >></a></p>
  </body>
 ```
Congratulations! You're ready to proceed to the next module: Managing Your Lab.
