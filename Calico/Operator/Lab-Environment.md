4. Creating the Lab Environment

To provision the lab environment we will be using cloud-init from Canonical to customize our virtual machines.

Quick start

If you’re on Linux, Mac, or have access to a Bash shell on Windows you can follow these steps to get up and running quickly:
```
curl https://raw.githubusercontent.com/tigera/ccol1/main/control-init.yaml | multipass launch -n control -m 2048M 20.04 --cloud-init -
curl https://raw.githubusercontent.com/tigera/ccol1/main/node1-init.yaml | multipass launch -n node1 20.04 --cloud-init -
curl https://raw.githubusercontent.com/tigera/ccol1/main/node2-init.yaml | multipass launch -n node2 20.04 --cloud-init -
curl https://raw.githubusercontent.com/tigera/ccol1/main/host1-init.yaml | multipass launch -n host1 20.04 --cloud-init -

```
Windows

If you’re on Windows, in powershell you can follow these steps to get up and running:
```
Invoke-WebRequest -UseBasicParsing 'https://raw.githubusercontent.com/tigera/ccol1/main/control-init.yaml' | Select-Object -Expand Content | multipass launch -n control -m 2048M 20.04 --cloud-init -
Invoke-WebRequest -UseBasicParsing 'https://raw.githubusercontent.com/tigera/ccol1/main/node1-init.yaml' | Select-Object -Expand Content | multipass launch -n node1 20.04 --cloud-init -
Invoke-WebRequest -UseBasicParsing 'https://raw.githubusercontent.com/tigera/ccol1/main/node2-init.yaml' | Select-Object -Expand Content | multipass launch -n node2 20.04 --cloud-init -
Invoke-WebRequest -UseBasicParsing 'https://raw.githubusercontent.com/tigera/ccol1/main/host1-init.yaml' | Select-Object -Expand Content | multipass launch -n host1 20.04 --cloud-init -
Starting the Instances
```
On some platforms, multipass requires you to start the VMs after they have been launched. We can do this by using the multipass start command.
```
multipass start --all

``
Throughout the deployments for the labs, the instances will reboot once provisioning is complete. As a result, you may have to wait a minute until the instance has fully provisioned. A quick way to check the current state of the cluster is to use the multipass list command.
```
multipass list
Example output:

Name                    State             IPv4             Image
control                 Running           172.17.78.3      Ubuntu 20.04 LTS
host1                   Running           172.17.78.6      Ubuntu 20.04 LTS
node1                   Running           172.17.78.7      Ubuntu 20.04 LTS
node2                   Running           172.17.78.12     Ubuntu 20.04 LTS
```
Validating the Environment

To validate the lab has successfully started after all four instances we will enter the host1 shell:
```
multipass shell host1

```
Once you reach the command prompt of host1, run kubectl get nodes.
```
kubectl get nodes -A

```
Example output:
```
NAME      STATUS     ROLES    AGE     VERSION
node1     NotReady   <none>   4m44s   v1.18.10+k3s1
node2     NotReady   <none>   2m48s   v1.18.10+k3s1
control   NotReady   master   6m36s   v1.18.10+k3s1
Note the “NotReady” status. This is because we have not yet installed a CNI plugin to provide the networking.
```

The instance we will be using for the following labs will be host1 unless otherwise specified. Think of host1 as your primary entry point into the kubernetes ecosystem, with the other instances acting as the cluster in the cloud.


