# Ingress Controller

<!--toc:start-->

- [Ingress Controller](#ingress-controller)
  - [Introduction](#introduction)
  <!--toc:end-->

## Introduction

We will explore the ingress controller, it is used to expose resources
outside the cluster.It defines the HTTP(S) routing to the services
inside the cluster.

It implements the layer 7 application load balancing(HTTP or application)
to access the application from the outside world.

Features:

- content based routing like host-based,host path etc
- ssl/tls certificate termination(http to https)

We will have ingress controller installed on the cluster as deployment or daemonset.
There are ingress resource(rules) and ingress controller.

So lets first clone the ingress controller from nginx github.

```bash
git clone https://github.com/nginx/kubernetes-ingress --branch=v3.0.2
```

After the change the directory to kubernetes-ingress folder.

We need to apply the following yaml files.

```bash
# create namespace and sa
kubectl apply -f common/ns-and-sa.yaml

# create rbac
kubectl apply -f rbac/rbac.yaml
# create default service secret
kubectl apply -f common/default-server-secret.yaml
kubectl apply -f common/nginx-config.yaml

kubectl apply -f config/crd/bases/k8s.nginx.org_virtualservers.yaml
kubectl apply -f config/crd/bases/k8s.nginx.org_virtualserverroutes.yaml
kubectl apply -f config/crd/bases/k8s.nginx.org_transportservers.yaml
kubectl apply -f config/crd/bases/k8s.nginx.org_policies.yaml
kubectl apply -f config/crd/bases/k8s.nginx.org_globalconfigurations.yaml
```
