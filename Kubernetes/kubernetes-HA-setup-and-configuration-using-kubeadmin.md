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

Launch the EC2 instance for HA proxy and first of all,we need to update
and upgrade the system packages.

```bash
sudo -i
apt update -y
apt upgrade -y
```

Install HA proxy:

```bash
sudo apt install -y haproxy
```

We need to update the HA proxy configuration file to include the control plane nodes
'/etc/haproxy/haproxy.cfg'. The configuration file should look like this:

```bash
# existing configuration

frontend kube-apiserver
        bind *:6443
        mode tcp
        option tcplog
        default_backend kube-apiserver

backend kube-apiserver
        mode tcp
        option tcplog
        option tcp-check
        balance roundrobin
        #default-server inter 10s downinter 5s rise 2 fall 2 slowstart 60s
        # maxconn 20
        # maxqueue 256 weight 100
        server kube-apiserver-1 <controplanode-ip>:6443 check
        server kube-apiserver-2 <controplanode-ip>:6443 check
        server kube-apiserver-3 <controplanode-ip>:6443 check

```

> [!NOTE]
> Here,we need to replace `<controplanode-ip>` with the actual IP addresses of
> the control plane nodes. We can also use the private IP addresses of the
> control plane nodes.

After updating the configuration file, we need to restart the HA proxy service:

```bash
systemctl enable haproxy
systemctl restart haproxy
```

After that we can check the status of the HA proxy service:

```bash
systemctl status haproxy
```

To check the connection from the control plane nodes to the HA proxy, we can
use `nc`
tool:

```bash
nc -v <haproxy-ip> 6443
```

After that we can proceed to setup the control plane nodes.

```bash
kubeadm init --control-plane-endpoint "172.31.90.220:6443" --upload-certs --pod-network-cidr 192.168.0.0/16
```

Then we can run the following command to set up the kubeconfig file for these.

```bash

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config


# Join the control plane nodes to the cluster
kubeadm join 172.31.90.220:6443 --token nwz7io.iflc3x3eamikgule \
        --discovery-token-ca-cert-hash sha256:62362eda95c45f9ee691b85949011a1e06e3d8ddc99849c65a3911003e9c02d1 \
        --control-plane --certificate-key 328a5bf38928b750c4ceba5b8021a472c3ae524e659b9ec5d07d33f6761058c5

# worker node join command
kubeadm join 172.31.90.220:6443 --token wdob3v.yhqqujemou350xel --discovery-token-ca-cert-hash sha256:62362eda95c45f9ee691b85949011a1e06e3d8ddc99849c65a3911003e9c02d1
```

After that we need to install the networking plugin for the cluster. We
are going to use calico as the networking plugin for the cluster. We can
install it using the following command:

[Documentation](https://docs.tigera.io/calico/latest/getting-started/kubernetes/k8s-single-node)

```bash
https://docs.tigera.io/calico/latest/getting-started/kubernetes/k8s-single-node
```
