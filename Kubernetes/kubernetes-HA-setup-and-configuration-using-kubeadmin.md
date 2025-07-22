# Kubernetes HA Setup and Configuration using Kubeadmin

<!--toc:start-->

- [Kubernetes HA Setup and Configuration using Kubeadmin](#kubernetes-ha-setup-and-configuration-using-kubeadmin)
  - [Introduction](#introduction)
  - [OS and Pre-Requisites](#os-and-pre-requisites)
  <!--toc:end-->

## Introduction

We are going to set up a Kubernetes cluster using Kubeadm and containerd tools. At
least 2GB of RAM and 2 cores of CPU are required to run a single-node cluster.

We are going to use AWS EC2 instances for this setup, but you can use any other cloud
provider as well.

Tools we are going to use:

- kubeadm: A tool to bootstrap Kubernetes clusters.
- containerd: A lightweight container runtime.
- calico: A CNI plugin for networking.
- kubelet: The primary node agent that runs on each node.
- kubectl: A command-line tool to interact with the Kubernetes cluster.
- ec2 instances: We will use AWS EC2 instances for the setup.
- HA proxy: A load balancer to distribute traffic across multiple control plane nodes.

Notes that we are going to create:
-1 node for HA proxy
-3 node for control plane(for high availability)
-1 or more compute node for running pods

## OS and Pre-Requisites

We are going to use Ubuntu 22.04 LTS as the operating system for the setup.We
need to disable swap memory for the system because it will slow down the performance
of the cluster.And minimum 16GB of Storage and 2vCPU and 4GB of RAM is required for
the cluster.
