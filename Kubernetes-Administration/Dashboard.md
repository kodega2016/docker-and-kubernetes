# Kubernetes Dashboard

<!--toc:start-->

- [Kubernetes Dashboard](#kubernetes-dashboard)
  - [Introduction](#introduction)
  - [Deploy the Dashboard](#deploy-the-dashboard)
  <!--toc:end-->

## Introduction

We can use the kuberentes dashboard to viusalize the kubernetes cluster.
It is a web-based UI that allows users to manage and monitor their Kubernetes
clusters.

We can also provide the read-only access to the dashboard for the users.

## Deploy the Dashboard

We need to deploy the dashboard first.To deploy we need to apply the following
manifest file.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```

We can view all the resources created by the dashboard using the following command.

```bash
kubectl get all -n kubernetes-dashboard
```

After that we need to create a service account and cluster role binding,we are using
the existing cluster-admin role for the dashboard.

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: admin-user
  namespace: kubernetes-dashboard
```

For the cluster role binding we can use the following manifest file.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: admin-user
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
  - kind: ServiceAccount
    name: admin-user
    namespace: kubernetes-dashboard
```

After creating the service account and cluster role binding we need to get the
token for the service account using the following command.

```bash
kubectl create token admin-user -n kubernetes-dashboard
```

[Deploy Kubernetes Dashboard](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)

```

```

```

```
