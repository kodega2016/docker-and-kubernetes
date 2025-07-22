# Kubernetes Setup using Kubeadm and containerd tools

<!--toc:start-->

- [Kubernetes Setup using Kubeadm and containerd tools](#kubernetes-setup-using-kubeadm-and-containerd-tools)
  - [Introduction](#introduction)
  - [Container and K8s Setup](#container-and-k8s-setup)
  - [Why containerd not docker?](#why-containerd-not-docker)
  - [Setup Steps](#setup-steps)
    - [Prepare the system to use containerd](#prepare-the-system-to-use-containerd)
    - [Installing the kubernetes tools](#installing-the-kubernetes-tools)
    <!--toc:end-->

## Introduction

We are going to setup kubernetes cluster using kubeadm and containerd tools,
At least 2GB of RAM and 2 cores of CPU are required to run a single node cluster.

Disable swap memory for the system because it will slow down the performance of the
cluster.

## Container and K8s Setup

- Install containerd for container runtime
- Install kubeadm,kubelet and kubectl for kubernetes cluster setup
- Setup calico as CNI plugin for networking
- Disable taints on the master node to allow it to run pods in a single node cluster

## Why containerd not docker?

The kubelets in kubernetes use containerd as the container runtime which is a lightweight
and educated version of docker. It is designed to be used in production environments,The
docker runtime also uses containerd for the containers it runs.So, we can use containerd
as the container runtime for kubernetes cluster setup.

There is CRI (Container Runtime Interface) which is used by kubelet to interact with
the container runtime in our case containerd.

## Setup Steps

First we need to install the ubuntu server on the virtual machine(UTM in our case)
and install ubuntu server,while installing ubuntu server we need to select the ipv4
should be configured manually and set the subnet,ip address and gateway for the static
ip address.

First we need to install ubuntu-server in utm with bridged network interface,
On the ipv4 configuration we need to set the static ip address,subnet and gateway.

| Network Interface | IPv4 Configuration |
| ----------------- | ------------------ |
| Subnet            | 192.168.1.0/24     |
| IP Address        | 192.168.1.100      |
| Gateway           | 192.168.1.1        |
| Nameserver        | 8.8.8.8,1.1.1.1    |

After installing ubuntu server,update and upgrade the system packages.

```bash
sudo apt update && sudo apt upgrade -y
```

### Prepare the system to use containerd

**_/etc/modules-load.d/containerd.conf_**
This file is used to load the required kernel modules for containerd.

```bash
cat > /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF
```

This loads the overlay and br_netfilter kernel modules which are required
for containerd to work properly.

overlay is a filesystem that allows multiple layers of filesystems
to be combined into a single filesystem,

br_netfilter is a kernel module that allows the network traffic to
be filtered by the container runtime using bridge networking.

After,that run the following commands to apply the changes:

```bash
modprobe overlay
modprobe br_netfilter
```

**_/etc/sysctl.d/99-kubernetes-cri.conf_**

```bash
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
```

This sets the persistent kernel parameters for the container runtime
which affects how the network traffic is handled by the container runtime.

`net.bridge.bridge-nf-call-iptables = 1`
This enables the iptables rules to be applied to the network traffic
using bridge networking.

`net.ipv4.ip_forward = 1`
This enables the IP forwarding for the system which is required
to allow the network traffic to be forwarded between the host and
containers.

`net.bridge.bridge-nf-call-ip6tables = 1`
same as above but for ipv6 traffic.

Apply the changes using sysctl command:

```bash
sysctl --system
```

### Installing the kubernetes tools

Installing the containerd runtime first.

```bash
apt-get update && apt-get install -y apt-transport-https ca-certificates curl software-properties-common

## Add Docker’s official GPG key
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | apt-key add -

## Add Docker apt repository.
add-apt-repository \
    "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
    $(lsb_release -cs) \
    stable"

## Install containerd
apt-get update && apt-get install -y containerd.io

# Configure containerd
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml
sed -i 's/SystemdCgroup = false/SystemdCgroup = true/g' /etc/containerd/config.toml

# Restart containerd
systemctl restart containerd

# To execute crictl CLI commands, ensure we create a configuration file as mentioned below
cat > /etc/crictl.yaml <<EOF
runtime-endpoint: unix:///run/containerd/containerd.sock
image-endpoint: unix:///run/containerd/containerd.sock
timeout: 2
EOF
```

After preparing the system to use containerd, we need to install
the kubeadm,kubelet and kubectl tools which are required to setup
the kubernetes cluster.

```bash
sudo apt update
sudo apt-get install -y apt-transport-https ca-certificates curl gpg

curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.32/deb/Release.key \
| sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg


echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.32/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

> [!NOTE]
> We need to disable the swap memory for the system because it will slow down
> the performance of the cluster.

Setup the cluster using kubeadm, we need to run the following command.

**_Initializing the kubeadm cluster_**

```bash
sudo kubeadm init --apiserver-advertise-address=192.168.1.100 \
  --cri-socket=unix:///run/containerd/containerd.sock \
  --pod-network-cidr=10.244.0.0/16
```

**_Setup kubectl_**

```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

Download the calico CNI plugin for networking in kubernetes cluster.

```bash
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
wget https://docs.projectcalico.org/manifests/custom-resources.yaml

or

Please refer the official document before proceeding further
https://projectcalico.docs.tigera.io/getting-started/kubernetes/quickstart

Note:
- Since I am using subnet for POD as 10.244.0.0/16 instead of default 192.168.0.0/16 which will conflict with my nodes ip address. First I downloaded custom-resources.yaml file and updates below parameter.
- cidr: 10.244.0.0/16


# After update, apply the YAML file
kubectl apply -f custom-resources.yaml
```

After that we can run the following command to check the status of the cluster.

```bash
kubectl get nodes
```

By default the node will be control plane node and it will be tainted so that it
will not allow to run any pods on it, we need to remove the taint from the node to
allow it to run pods on it.

After successful setup of the control plan node,clone the virtual machine into one
for the compute plan node.

Change the hostname of the compute node using the following command.

```bash
hostnamectl set-hostname computeplanenodeone
logout
```

After that we need to change the ip address of the compute node,so lets update the
netplan file to change the ip address of the compute node.

```bash
sudo vim /etc/netplan/500-cloud-init.yaml
```

Update the ip address of the compute node to `192.168.1.101`

Reset the cluster on the control plane node using the following command.

```bash

# reset the kubeadm cluster
sudo kubeadm reset --cri-socket=unix:///run/containerd/containerd.sock

sudo rm -rf /etc/kubernetes/
sudo rm -rf ~/.kube/
sudo systemctl restart kubelet
sudo systemctl restart containerd

# create the cluster again
sudo kubeadm init --apiserver-advertise-address=192.168.1.100 \
  --cri-socket=unix:///run/containerd/containerd.sock \
  --pod-network-cidr=10.244.0.0/16

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


# install networking plugin
kubectl create -f https://raw.githubusercontent.com/projectcalico/calico/v3.30.2/manifests/tigera-operator.yaml

# join the compute node to the cluster
kubeadm join 192.168.1.100:6443 \
  --token 7el0yk.uar5sr1a2vis9qbx \
  --discovery-token-ca-cert-hash sha256:ec6fbbc0a9befbd4759e2825600b0333d37a852fab6484be7362e4904e816875
```

After that we can check the availability of the nodes in the cluster using the following
command.

```bash
kubectl get nodes
```
