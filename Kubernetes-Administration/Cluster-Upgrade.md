# Kubernetes Cluster Upgrade

<!--toc:start-->

- [Kubernetes Cluster Upgrade](#kubernetes-cluster-upgrade)
  - [Introduction](#introduction)
  - [Upgrade Kubernetes Cluster](#upgrade-kubernetes-cluster)
  <!--toc:end-->

## Introduction

We need to upgrade the primary control plane node first,followed
by the secondary control plane nodes, and finally the worker nodes.

Upgrade one by one, waiting for each node to be fully operational
before proceeding to the next.

We make sure to take backup for the etcd database before starting
the upgrade process if in case of any issues.

All the containers will be restarted since the container hash
value gets changed after the upgrade.

The CNI should be upgraded manually based on the CNI provider.

The kubeadm store the manifests and configuration files
under the
`/etc/klubernetes/tmp/kubeadm-backup-etcd-<date>-<time>`
`/etc/klubernetes/tmp/kubeadm-backup-manifests-<date>-<time>`
directory.

## Upgrade Kubernetes Cluster

We need to run the following commands on all the nodes.

```bash
sudo apt-get update
```

After the check the available version of kubeadm.

```bash
sudo apt-cache madison kubeadm
```

We want to unhold the kubeadm package if it is held.

```bash
sudo apt-mark unhold kubeadm
```

Then upgrade the kubeadm package.

```bash
sudo apt-get install -y kubeadm=1.27.3-00
```

After the upgrade check the version of kubeadm.

```bash
sudo kubeadm version
# hold the kubeadm package
sudo apt-mark hold kubeadm
```

We can check the upgrade plan to see the available versions.

```bash
kubeadm upgrade plan
kubeadm upgrade apply v1.32.8
```

After this commmand,we need to drain the node before upgrading
the kubelet and kubectl packages.

```bash
kubectl drain <node-name> --ignore-daemonsets
```

which safely evicts all the pods from the node for
the upgrade process.The ignore-daemonsets flag ensures
that the daemonset-managed pods are not evicted.

After that unhold the kubelet and kubectl packages.

```bash
sudo apt-mark unhold kubelet kubectl
```

Then upgrade the kubelet and kubectl packages.

```bash
sudo apt-get install -y kubelet=1.27.3-00 kubectl=1.27.3-00
```

After that again hold the kubelet and kubectl packages.

```bash
sudo apt-mark hold kubelet kubectl
```

Then restart the kubelet service to apply the changes.

```bash
systemctl daemon-reload
systemctl restart kubelet
```

After that uncordon the node to make it schedulable again.

```bash
kubectl uncordon <node-name>
```

Finally verify the node status to ensure it is ready and
properly functioning.

```bash
kubectl get nodes
```
