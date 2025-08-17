# Storage in Kubernetes

<!--toc:start-->

- [Storage in Kubernetes](#storage-in-kubernetes)
  - [Introduction](#introduction)
  - [Kubernetes Volumes](#kubernetes-volumes)
  - [EmptyDir Volume](#emptydir-volume)
  - [hostPath Volume](#hostpath-volume)
  - [NFS server for kubernetes](#nfs-server-for-kubernetes)
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

## hostPath Volume

It is used to mount a file or directory from the host node's file system into a pod.
So it is useful for sharing the files between the host and the pod.

It is useful for node specific configurations or for accessing host resources.

If the pod is rescheduled to another node, the hostPath volume will not be available
for that pods, so it is not suitable for persistent storage.

It exists beyond the life-cycle of the pod.

## NFS server for kubernetes

It is a network file system that allows multiple pods and nodes to share the same
file and directory structure. It is useful for sharing files between pods and nodes.

It is great for persistent storage as it allows multiple pods to access the
same data.

So lets first create a nfs server in virtual machine.We need to run the following
commands.

```bash
sudo apt update
sudo apt upgrade -y
sudo apt install nfs-kernel-server -y
```

After installing we can check the status for that service using the following command.

```bash
systemctl status nfs-kernel-server
```

After that we need to make a folder to expose for nfs server.

```bash
mkdir /var/nfs
```

Now make the folder ownership to anonymous user and group.

```bash
chown nobody:nogroup /var/nfs
```

We can verify the ownership of the folder using the following command.

```bash
ls -ld /var/nfs
```

After that add new line to the `/etc/exports` file to export the folder.

```bash
# add this line to the file
*(rw,sync,no_subtree_check,no_root_squash)
```

We need to run the following command to export the folder.

```bash
sudo exportfs -ra
```

We must install the `nfs-common` package on the client machine to mount the NFS share.

```bash
sudo apt install nfs-common -y
```

We can verify the mount using the following command.

```bash
mount | grep nfs
```

## Jenkins CI CD setup with nfs server

We need to create a nfs service folder in the nfs server machine.

```bash
mkdir /var/nfs/jenkins-master
```

And we need to give ownership to the folder.

```bash
sudo chown 1000:1000 /var/nfs/jenkins-master
```
