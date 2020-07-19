
# Etcd 

- Kubernetes uses etcd to store all its data – its configuration data, its state, and its metadata. Kubernetes is a distributed system, so it needs a
distributed data store like etcd. etcd lets any of the nodes in the Kubernetes cluster read and write data.

- Kubernetes is a set of processes running on several machines.(Actual, literal computer processes, not business-speak processes.) 
One of these machines is the master and the rest are the worker nodes.


# What is etcd?

- Now, we know that Kubernetes is distributed – it runs on several machines at the same time.

- So, it needs a distributed database. One that runs on several machines at the same time. One that makes it easy to store data across a 
cluster and watch for changes to that data.

# How does Kubernetes use etcd?

- Kubernetes uses etcd as a key-value database store. It stores the configuration of the Kubernetes cluster in etcd.

- It also stores the actual state of the system and the desired state of the system in etcd.

- It then uses etcd’s watch functionality to monitor changes to either of these two things. If they diverge, Kubernetes makes changes to reconcile the actual state and the desired state.

# A Kubernetes cluster stores all its data in etcd.

- Anything you might read from a `kubectl get xyz` command is stored in etcd.

- Any change you make via `kubectl create` will cause an entry in etcd to be updated.

- Any node crashing or process dying causes values in etcd to be changed.

- The set of processes that make up Kubernetes use etcd to store data and notify each other of changes.


# Download and Install the etcd Binaries

# Download the official etcd release binaries from the coreos/etcd GitHub project

```
wget -q --show-progress --https-only --timestamping \
  "https://github.com/etcd-io/etcd/releases/download/v3.4.10/etcd-v3.4.10-linux-amd64.tar.g
  
 ```
 # Extract and install the etcd server and the etcdctl command line utility:
 ```
 tar -xvf etcd-v3.4.10-linux-amd64.tar.gz
  sudo mv etcd-v3.4.10-linux-amd64/etcd* /usr/local/bin/

 ```
 # Configure the etcd Server
 ```
 sudo mkdir -p /etc/etcd /var/lib/etcd
  sudo chmod 700 /var/lib/etcd
  sudo cp ca.pem kubernetes-key.pem kubernetes.pem /etc/etcd/
 
 
 ```
 the instance internal IP address will be used to serve client requests and communicate with etcd cluster peers. Retrieve the internal IP address for the current compute instance:
 ```
INTERNAL_IP=$(curl -s -H "Metadata-Flavor: Google" \
  http://metadata.google.internal/computeMetadata/v1/instance/network-interfaces/0/ip)

 
 
 ```
 Each etcd member must have a unique name within an etcd cluster. Set the etcd name to match the hostname of the current compute instance:
 ```
 TCD_NAME=$(hostname -s)
 
 
 ```
 Create the etcd.service systemd unit file:
 ```
 cat <<EOF | sudo tee /etc/systemd/system/etcd.service
[Unit]
Description=etcd
Documentation=https://github.com/coreos

[Service]
Type=notify
ExecStart=/usr/local/bin/etcd \\
  --name ${ETCD_NAME} \\
  --cert-file=/etc/etcd/kubernetes.pem \\
  --key-file=/etc/etcd/kubernetes-key.pem \\
  --peer-cert-file=/etc/etcd/kubernetes.pem \\
  --peer-key-file=/etc/etcd/kubernetes-key.pem \\
  --trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-trusted-ca-file=/etc/etcd/ca.pem \\
  --peer-client-cert-auth \\
  --client-cert-auth \\
  --initial-advertise-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-peer-urls https://${INTERNAL_IP}:2380 \\
  --listen-client-urls https://${INTERNAL_IP}:2379,https://127.0.0.1:2379 \\
  --advertise-client-urls https://${INTERNAL_IP}:2379 \\
  --initial-cluster-token etcd-cluster-0 \\
  --initial-cluster controller-0=https://10.240.0.10:2380,controller-1=https://10.240.0.11:2380,controller-2=https://10.240.0.12:2380 \\
  --initial-cluster-state new \\
  --data-dir=/var/lib/etcd
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target
EOF
 
 
 
 ```
 Start the etcd Server
 ```

 sudo systemctl daemon-reload
  sudo systemctl enable etcd
  sudo systemctl start etcd
 
 ```
 Remember to run the above commands on each controller node: controller-0, controller-1, and controller-2.
 
 Verification

List the etcd cluster members:

```
udo ETCDCTL_API=3 etcdctl member list \
  --endpoints=https://127.0.0.1:2379 \
  --cacert=/etc/etcd/ca.pem \
  --cert=/etc/etcd/kubernetes.pem \
  --key=/etc/etcd/kubernetes-key.pem


```
     3a57933972cb5131, started, controller-2, https://10.240.0.12:2380, https://10.240.0.12:2379, false
     f98dc20bce6225a0, started, controller-0, https://10.240.0.10:2380, https://10.240.0.10:2379, false
     ffed16798470cab5, started, controller-1, https://10.240.0.11:2380, https://10.240.0.11:2379, false

