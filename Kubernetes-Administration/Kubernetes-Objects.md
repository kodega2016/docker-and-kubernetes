# Kubernetes Objects Overview

<!--toc:start-->
- [Kubernetes Objects Overview](#kubernetes-objects-overview)
  - [What is a Pod?](#what-is-a-pod)
  - [What is initContainers?](#what-is-initcontainers)
<!--toc:end-->

In Kubernetes,each of the components is represented as an object.We
need to create these objects to run our applications in the kubernetes
cluster.

Workload objects:

- pods
- replicaset or replication controller
- deployments
- daemonsets
- statefulsets
- jobs or cronjobs

Some of the other objects are:

- services
- configmaps
- secrets
- persistent volumes
- persistent volume claims
- network policies
- namespaces
- ingress

## What is a Pod?

It is one of the smallest unit in the kubernetes,we can run multiple
containers in pod,normally related containers are running on the single
pod,they are also known as side car containers.

The pod is the logical unit for the shared volumes and shared networking
between containers,they can communicate with using the localhost.

We can have resources limits and requests for the containers.The
containers can have memory and cpu requests and limits.

## What is initContainers?

They are the special containers,they runs before the main containers
starts.They are used for the initialization tasks such as setting up
scripts,Each initContainers starts and finish and others containers starts.

initContainers can have their own images and configurations.
