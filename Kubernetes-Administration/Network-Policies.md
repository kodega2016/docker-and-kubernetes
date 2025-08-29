# Network Policies

<!--toc:start-->

- [Network Policies](#network-policies)
  - [Introduction](#introduction) - [Igress Type ipBlock](#igress-type-ipblock)
  <!--toc:end-->

## Introduction

Network policies in Kubernetes are a way to control the network traffic between
pods and services,They allow you to define rules that specify which pods can
communicate with each other and which cannot.

By default, all pods in a Kubernetes cluster can communicate with each other.

So, each of the pods gets its own IP address, and all pods can reach each other
without any restrictions.

Network polices supports two types of traffic:

- Ingress: Incoming traffic to a pod.
- Egress: Outgoing traffic from a pod.

There are default rules such as deny all ingress traffic and allow all egress
for pods in a namespace when a network policy is applied.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: default
spec:
  podSelector: {} # applies to all Pods in namespace
  policyTypes:
    - Ingress
```

Ingress and egress rules can be defined using labels, IP blocks, and ports.
ingress supports from and egress supports to.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: test-network-policy
  namespace: default
spec:
  podSelector:
    matchLabels:
      role: db
  policyTypes:
    - Ingress
    - Egress
  ingress:
    - from:
        - ipBlock:
            cidr: 172.17.0.0/16
            except:
              - 172.17.1.0/24
        - namespaceSelector:
            matchLabels:
              project: myproject
            podSelector:
              matchLabels:
                role: frontend
        - podSelector:
            matchLabels:
              role: frontend
      ports:
        - protocol: TCP
          port: 6379
  egress:
    - to:
        - ipBlock:
            cidr: 10.0.0.0/24
      ports:
        - protocol: TCP
          port: 5978
```

### Igress Type ipBlock

We can get the range of IP CIDR for the each node using the following command:

```bash
kubectl get ipamblocks.crd.projectcalico.org \
  -o jsonpath='{range .items[*]}{.spec.affinity}{" => "}{.spec.cidr}{"\n"}{end}'

```

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-workers-except-master
  namespace: default
spec:
  podSelector: {} # applies to all pods in namespace
  policyTypes:
    - Ingress
  ingress:
    - from:
        - ipBlock:
            cidr: 10.244.0.0/16 # allow the entire pod CIDR range
            except:
              - 10.244.235.192/26 # exclude master’s pod range
```

We also can have the selector for namespace and pods.

```yaml
- namespaceSelector:
    matchLabels:
      purpose: test
  podSelector:
    matchLabels:
      role: frontend
```

This rule allows traffic from pods with the label role=frontend in
the namespace with the label purpose=test.
