# Persistent Volume and Persistent Volume Claim

<!--toc:start-->

- [Persistent Volume and Persistent Volume Claim](#persistent-volume-and-persistent-volume-claim)
  - [Introduction](#introduction)
  - [Persistent Volume (PV)](#persistent-volume-pv)
  - [Persistent Volume Claim (PVC)](#persistent-volume-claim-pvc)
  - [Persistent Volume Access Modes](#persistent-volume-access-modes)
  <!--toc:end-->

## Introduction

In this section,we will learn about Persistent Volume (PV) and Persistent Volume
Claim (PVC) in Kubernetes.

Also we will explore storage class,volume expansion and volume snapshot.

By default,kuberenets volumes are ephemeral and tied to the lifecycle of a pod.
if the pod is deleted, the data in the volume is lost.

So, to persist data across pod restarts, we use Persistent Volumes (PV) and
Persistent Volume Claims (PVC).

## Persistent Volume (PV)

It is a piece of storage in the cluster that has been provisioned by an administrator
or dynamically provisioned using Storage Classes.

The PV is a separate resource in the cluster and is not tied to any specific pod.so
that it can be reused by different pods.

There are two modes of PV:

- File system (NFS, GlusterFS, etc.)
- Block storage (AWS EBS, GCE PD, Azure Disk, etc.)

It is in cluster level and can be used by any namespace in the cluster.

## Persistent Volume Claim (PVC)

It is a request for storage by a user. It is similar to a pod in that it specifies
the size and access mode of the storage required.

When a PVC is created, Kubernetes will look for a PV that matches the request.
If a matching PV is found, it will be bound to the PVC and the storage will be available

It requires the size of the storage and access mode
(ReadWriteOnce, ReadOnlyMany, or ReadWriteMany).

It is namespace scoped, meaning it can only be used by the namespace in which it
is created.When a PVC is created, Kubernetes will look for a PV that matches the
request.

## Persistent Volume Access Modes

**_ReadWriteOnce_**
It we need to give read and write access to pods running on a single node,this mode
is not suitable for multi-node clusters.

**_ReadWriteMany_**
If we need to give read and write access to pods running on multiple nodes,this
mode is suitable for multi-node clusters.

**_ReadOnlyMany_**
Pods running on multiple nodes can only read the data in the volume, but cannot write
to it.

**_ReadWriteOncePod_**
In this mode, a volume can be mounted as read-write by a single pod. This is useful
for scenarios where a volume needs to be accessed by a single pod, but that pod
needs to be able to write to the volume.

## PV and PVC Management

After the pvc is deleted, the pv will be in Released state.so we cannot use the same
pvc again until we make the pv available again.

We can do this by editing the pv and changing the status to Available.

```bash
vim /etc/kubernetes/pv.yaml
# where we need to remove the claimRef section
```

After removing the claimRef section,we got the pv in Available state.
