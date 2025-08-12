# Storage in Kubernetes

<!--toc:start-->

- [Storage in Kubernetes](#storage-in-kubernetes)
  - [Introduction](#introduction)
  - [Kubernetes Volumes](#kubernetes-volumes)
  <!--toc:end-->

## Introduction

We are going to explore the storage options available in Kubernetes, including persistent
volumes, persistent volume claims, and storage classes.

By default,the pods in kubernetes are ephemeral,meaning that we lose all the
data when the pods is deleted or replaced. To persist data, we need to use persistent
storage.

## Kubernetes Volumes

In general,containers are ephemeral, meaning that they can be created and destroyed
at any time.so that we may lose the data stored in the containers.so if we want to
persist the data, we need to use volumes.

And also if we want to share the data between the containers in the same pods,we
need to use volumes.

## EmptyDir Volume

It is one of the simplest volume types in Kubernetes. It is created when a pod is
created and deleted when the pod is deleted. It is stored in the node's filesystem
so it is not persistent across pod restarts. It is useful for temporary storage
such as caching or scratch space.
