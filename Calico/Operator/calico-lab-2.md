Installing the Sample Application

If you are not already on host1, you can enter host1 by using the multipass shell command.
```
multipass shell host1

```
To install yaobank into your kubernetes cluster, apply the following manifest:
```
kubectl apply -f https://raw.githubusercontent.com/tigera/ccol1/main/yaobank.yaml
```
