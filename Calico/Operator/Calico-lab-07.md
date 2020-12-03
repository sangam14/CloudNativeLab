to simulate a compromise of the customer pod we will exec into the pod and attempt to access the database directly from there.

Enter the customer pod

First we will find the customer pod name and store it in an environment variable to simplify future commands.

CUSTOMER_POD=$(kubectl get pods -n yaobank -l app=customer -o name)
Note that the CUSTOMER_POD environment variable only exists within your current shell, so if you exit that shell you must set it again in your new shell using the same command as above.

Now we will exec into the customer pod, and run bash, to give us a command prompt within the pod:

kubectl exec -it $CUSTOMER_POD -n yaobank -c customer -- /bin/bash
Access the database

From within the customer pod, we will now attempt to access the database directly, simulating an attack.  As the pod is not secured with NetworkPolicy, the attack will succeed and the balance of all users will be returned.

curl http://database:2379/v2/keys?recursive=true | python -m json.tool
Leaving the customer pod

To return from the pod back to our original host command line shell, use the exit command:

exit
