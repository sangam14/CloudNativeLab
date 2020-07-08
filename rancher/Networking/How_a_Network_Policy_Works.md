---
layout: default
title: How a Network Policy Works
parent: Rancher Networking
nav_order: 9
---


# How a Network Policy Works

In addition to the fields used by all Kubernetes manifests, the specification of the NetworkPolicy resource requires some extra fields.

# PODSELECTOR
This field tells Kubernetes how to find the Pods to which this policy applies. Multiple network policies can select the same set of Pods, and the ingress rules are 
applied sequentially. The field is not optional, but if the manifest defines a key with no value, it applies to all Pods in the namespace.

# POLICYTYPES

This field defines the direction of network traffic to which the rules apply. If missing, Kubernetes interprets the rules and only applies them to ingress traffic unless egress rules also appear in the rules list. This default 
interpretation simplifies the manifest's definition by having it adapt to the rules defined later.
Because Kubernetes always defines an ingress policy if this field is unset, a network policy for `egress-only` rules must explicitly define the `policyType` of Egress.


# EGRESS

Rules defined under this field apply to egress traffic from the selected Pods to destinations defined in the rule. Destinations can be an IP block (ipBlock), one or more Pods (podSelector), one or more namespaces (namespaceSelector), 
or a combination of both `podSelector` and `nameSpaceSelector`.
The following rule permits traffic from the Pods to any address in 10.0.0.0/24 and only on TCP port 5978:

```
egress: 
  - to:   
    - ipBlock:        
        cidr: 10.0.0.0/24    
     ports:   
       - protocol: TCP     
         port: 5978
```

The next rule permits outbound traffic for Pods with the labels app=myapp and role=backendto any host on TCP or UDP port 53:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: name: db-egress-denyall
spec: 
podSelector:  
  matchLabels:     
    app: myapp    
    role: backend 
policyTypes:
- Egress 
egress: 
 - ports:  
 - port: 53   
     protocol: UDP  
     - port: 53    
     protocol: TCP

```

Egress rules work best to limit a resource’s communication to the other resources on which it relies. If those resources are in a specific block of IP addresses, 
use the ipBlock selector to target them, specifying the appropriate ports:

```
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata: name: db-egress-denyall
spec: 
podSelector:  
  matchLabels:     
    app: myapp    
    role: backend 
policyTypes:
- Egress 
egress: 
 - ports:  
 - port: 53   
     protocol: UDP  
     - port: 53    
     protocol: TCP
- to:  
  - ipBlock:      
      cidr: 10.0.0.0/24  
    ports:  
    - protocol: TCP    
      port: 3306
```

# INGRESS
Rules listed in this field apply to traffic that is inbound to the selected Pods. If the field is empty, all inbound traffic will be blocked. 
The example below permits inbound access from any address in 172.17.0.0/16 unless it’s within 172.17.1.0/24. It also permits traffic from any Pod in 
the namespace myproject.(Note the subtle distinction in how the rules are listed. Because namespaceSelector is a separate item in the list, it matches with an 
or value. Had namespaceSelector been listed as an additional key in the first list item, it would permit traffic that came from the specified ipBlock and was also from the namespace myproject.)

```

ingress:
  - from:  
  - ipBlock:    
      cidr: 172.17.0.0/16    
      except:     
        - 172.17.1.0/24  
  - namespaceSelector:   
     matchLabels:       
       project: myproject  
  - podSelector:     
       matchLabels:       
         role: frontend  
   ports: 
      - protocol: TCP    
        port: 6379



```

The next policy permits access to the Pods labeled app=myapp and role=web from all sources, external or internal.

```

kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: web-allow-all-access
spec: 
podSelector:  
   matchLabels:    
     app: myapp     
     role: web 
ingress:
- from: []


```
Consider, however, that this allows traffic to any port on those Pods. Even if no other ports are listening, the principle of least privilege states that we only want to expose what we need to expose for the services to work. The following modifications to the NetworkPolicy take this rule into account by only allowing inbound traffic to the ports where our Service is running.

```
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata: name: web-allow-all-access-specific-port
spec: 
podSelector: 
 matchLabels:  
  app: myapp    
   role: web 
ingress: 
- ports:   
  - port: 8080   
from: []

```
Apart from opening incoming traffic on certain ports, you can also enable all traffic from a set of Pods inside the cluster. This enables a few trusted applications to reach out from the application on all ports and is especially useful when workloads in a cluster communicate with each other over many random ports. The opening of traffic from certain Pods is achieved by using labels as described in the policy below:

```
kind: NetworkPolicyapi
Version: networking.k8s.io/v1
metadata: name: web-allow-internal-port80
spec:
  podSelector:  
    matchLabels:   
      app: "myapp"    
      role: "web" 
ingress: 
- ports: 
  - port: 8080  
from:   
- podSelector:  
    matchLabels:       
      app: "mytestapp"     
      role: "web-test-client"
```

Even if a Service listens on a different port than where the Pod’s containers listen, use the container ports in the network policy. Ingress rules affect inter-Pod communication, and the policy does not know about the abstraction of the service.
