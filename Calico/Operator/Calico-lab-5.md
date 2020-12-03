 you already have the "Yet Another Online Bank" (yaobank) installed, as created in the “Installing the Sample Application” module in Week 1.

If you don’t already have it installed then go back and do so now.

For reference, this is architecture of YAOBank:

webui(customer)---> bussiness logic(summery)----> Data ----> (database)

If you are not already on host1, you can enter host1 by using the multipass shell command.
```
multipass shell host1
```
You can validate the yaobank installation by using kubectl get pods.
```
kubectl get pods -n yaobank
Example output:

NAME                        READY   STATUS    RESTARTS   AGE
database-6c5db58d95-slsph   1/1     Running   0          26m
summary-85c56b76d7-28chk    1/1     Running   0          26m
summary-85c56b76d7-l7rsv    1/1     Running   0          26m
customer-574bd6cc75-5cwnn   1/1     Running   0          26m
```
