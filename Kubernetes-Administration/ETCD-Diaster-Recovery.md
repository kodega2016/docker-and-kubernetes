# ETCD Diaster Recovery and Backup

<!--toc:start-->

- [ETCD Diaster Recovery and Backup](#etcd-diaster-recovery-and-backup)
  - [Introduction](#introduction)
  - [ETCD Cli Tools Setup](#etcd-cli-tools-setup)
  - [Backup ETCD Data](#backup-etcd-data)
  - [Restore ETCD Data](#restore-etcd-data)
  <!--toc:end-->

## Introduction

ETCD is a distributed key-value store that is used as the primary data store
for Kubernetes. It is crucial to ensure that ETCD data is backed up regularly and
that you have a disaster recovery plan in place in case of data loss or corruption.

We can check the etcd version running in out cluster by running the following command:

```bash
kubectl exec -n kube-system -it etcd-k8s-master-1 -- etcdctl version
```

We can also check the etcd client version by running the following command:

```bash
kubectl exec -n kube-system -it etcd-k8s-master-1 -- etcdctl --version
```

## ETCD Cli Tools Setup

After checking the etcd version, we can download the etcd cli tools from the official
github repository. In this case, we are using etcd version 3.5.1.

```bash
wget https://github.com/etcd-io/etcd/releases/download/v3.5.1/etcd-v3.5.1-linux-arm64.tar.gz
```

After that unzip the tar file:

```bash
tar -zxvf etcd-v3.5.1-linux-arm64.tar.gz
```

Then move the etcdctl binary to /usr/local/bin directory:

```bash
sudo mv etcd-v3.5.1-linux-arm64/etcdctl /usr/local/bin/
```

After that we can check the etcdctl version by running the following command:

```bash
etcdctl version
```

## Backup ETCD Data

There are two connections, one is peer connection and the other is client connection.
The default port for peer connection is 2380 and for client connection is 2379.

We need to connect to the peer connection to take a backup of the etcd data.

After that lets get the ip address of the etcd server by running the following command:

```bash
kubectl get pods -n kube-system -o wide | grep etcd
```

It is normally the master node ip address.so lets set it into a variable:

```bash
export ENDPOINT=https://192.168.65.5:2380
```

After that run the following command to view the status of the etcd cluster:

```bash
ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  endpoint status --write-out=table
```

To take snapshot of the etcd data run the following command:

```bash
ETCDCTL_API=3 etcdctl \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  snapshot save /tmp/snapshot.db
```

We can now verify the snapshot by running the following command:

```bash
sudo ETCDCTL_API=3 etcdctl snapshot status /tmp/snapshot.db
# to print the table format
sudo ETCDCTL_API=3 etcdctl snapshot status /tmp/snapshot.db --write-out=table
```

## Restore ETCD Data

We can check all the configurations of the etcd server by running the following command:

```bash
cat /etc/kubernetes/manifests/etcd.yaml
```

From here, we need the following information to restore the etcd data:

- data-dir
- initial-cluster
- name
- initial-cluster-token
- initial-advertise-peer-urls

So first we need to delete the existing data-dir:

```bash
rm -rf /var/lib/etcd
```

After that we can restore the etcd data from the snapshot we took earlier.

```bash
etcdutl snapshot restore snapshot.db \
  --name k8s-master \
  --data-dir /var/lib/etcd \
  --initial-cluster k8s-master=https://192.168.65.5:2380 \
  --initial-cluster-token etcd-cluster-1 \
  --cacert=/etc/kubernetes/pki/etcd/ca.crt \
  --cert=/etc/kubernetes/pki/etcd/server.crt \
  --key=/etc/kubernetes/pki/etcd/server.key \
  --initial-advertise-peer-urls http://host1:2380
```
