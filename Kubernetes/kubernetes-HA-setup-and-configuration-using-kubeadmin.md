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

We are going to use AWS EC2 instances for this setup.

Ec2 instance specification:

- Instance Type: t2.medium
- OS: Ubuntu 22.04 LTS
- Storage: 20GB
- Numer of Instances: 6
  - 1 for HA proxy
  - 3 for control plane nodes
  - 2 for compute nodes

## Install Docker

After installing the OS,we need to update and upgrade the system packages.

```bash
sudo -i
apt update -y
apt upgrade -y
# graceful reboot the system
init 6
```

Disable swap memory:

```bash
swapoff -a
```

Install containerd:

```bash
# Add Docker's official GPG key:
sudo apt-get update
sudo apt-get install ca-certificates curl
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

# Add the repository to Apt sources:
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "${UBUNTU_CODENAME:-$VERSION_CODENAME}") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
sudo apt-get update
sudo apt-get install containerd.io
```

Load the required modules for containerd:

```bash
cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter
```

Setup required sysctl params, these persist across reboots.

```bash
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system
```

Configure containerd

```bash
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml
```

Restart containerd

```bash
systemctl restart containerd
```

To execute crictl CLI commands, ensure we create a configuration file as
mentioned below

```bash
cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
EOF
```

Install Kubernetes tools

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

Download the Google Cloud public signing key:

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.29/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```

Create a new sources list file for Kubernetes:

```bash
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.29/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```

```bash
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Setup HA Proxy
