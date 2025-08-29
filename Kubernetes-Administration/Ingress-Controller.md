# Ingress Controller

<!--toc:start-->

- [Ingress Controller](#ingress-controller)
  - [Introduction](#introduction)
  - [Deploy Ingress resources for k8s(host based routing)](#deploy-ingress-resources-for-k8shost-based-routing)
    - [TLS certificate for ingress](#tls-certificate-for-ingress)
  - [Path Based Routing](#path-based-routing)
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

## Deploy Ingress resources for k8s(host based routing)

### TLS certificate for ingress

First of all lets generate self signed tls certificate with the following
command.

```bash
# for domain homeapp.kodega-ops.com
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -out homeapp-ingress-tls.crt \
  -keyout homeapp-ingress-tls.key \
  -subj "/CN=homeapp.kodega-ops.com/O=kodega"


# for domain homeworker.kodega-ops.com
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
  -out homeworker-ingress-tls.crt \
  -keyout homeworker-ingress-tls.key \
  -subj "/CN=homeworker.kodega-ops.com/O=kodega"
```

## Path Based Routing

We can do path based routing also,
