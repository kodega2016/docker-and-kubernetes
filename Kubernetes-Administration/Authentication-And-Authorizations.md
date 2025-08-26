# Authentication and Authorization in Kubernetes

<!--toc:start-->

- [Authentication and Authorization in Kubernetes](#authentication-and-authorization-in-kubernetes)
  - [Introduction](#introduction)
    - [Authentication Methods](#authentication-methods)
    - [Authorization Methods](#authorization-methods)
  - [Kubernetes Authentication Strategies](#kubernetes-authentication-strategies)
  - [Kubernetes Authorization Strategies](#kubernetes-authorization-strategies)
    - [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
  - [Understanding the kubeconfig file](#understanding-the-kubeconfig-file)
  - [Certificate Based Authentication](#certificate-based-authentication)
  - [Role and RoleBinding](#role-and-rolebinding) - [ClusterRole and ClusterRoleBinding](#clusterrole-and-clusterrolebinding)
  <!--toc:end-->

## Introduction

This document provides an overview of how authentication and authorization work in
Kubernetes, along with best practices for securing your cluster.

The authentication verifies the identity of a user or service, while authorization
verifies what actions that user or service is allowed to perform.

### Authentication Methods

There are several methods for authenticating to a Kubernetes cluster:

- Certificates: Client certificates can be used to authenticate users and services.
- Tokens: Bearer tokens can be used for authentication, often in conjunction with
- Usernames and Passwords: Basic authentication using usernames and
  passwords(LDAP, etc).

### Authorization Methods

It is the process of determining what actions a user or service can perform.
There are several authorization methods in Kubernetes:

- Role-Based Access Control (RBAC): RBAC is the most common method for authorization
  in Kubernetes.It allows you to define roles and bind them to users or
  service accounts.

- Attribute-Based Access Control (ABAC): ABAC allows you to define policies based
  the on user attributes, such as group membership or labels.

- Webhook: Webhook authorization allows you to delegate authorization decisions to
  the external service.

- Node Authorization: Node authorization is a built-in authorization mode that
  restricts what actions nodes can perform.

## Kubernetes Authentication Strategies

There are several strategies for authenticating to a Kubernetes cluster:

- Service Accounts
- Normal User Accounts

**_Service Accounts_**
These are special accounts used by applications running in pods to interact with
the kubernetes API. Each namespace has its own set of service accounts.

We can create service account and bind it to a role using the following commands:

```bash
kubectl create serviceaccount my-service-account -n my-namespace
kubectl create rolebinding my-role-binding \
--role=my-role \
--serviceaccount=my-namespace:my-service-account \
-n my-namespace
```

In order to implement user name and password,we need to enable the LDAP authentication
mechanism in the API server configuration.

The normal user can be the following:

- certificates based authentication
- token based authentication
- username and password authentication

## Kubernetes Authorization Strategies

There are following strategies for authorizing actions in a Kubernetes cluster:

- Attribute-Based Access Control (ABAC)
  It is a JSON file that defines policies based on user attributes.

- Role-Based Access Control (RBAC)
  It is the most common method for authorization in Kubernetes.Where we create roles
  and bind them to users or service accounts.It is called role binding.

- Node Authorization
  It is a built-in authorization mode that restricts what actions nodes can perform.
  by the kubelet.

- Webhook
  It allows you to delegate authorization decisions to an external service.

### Role-Based Access Control (RBAC)

There are four main components in RBAC:

- Role: A set of permissions that can be assigned to users or service accounts.
- ClusterRole: Similar to a Role, but it applies to the entire cluster.
- RoleBinding: Binds a Role to a user or service account within a specific namespace.
- ClusterRoleBinding: Binds a ClusterRole to a user or service account across the

> [!NOTE]
> Role is scoped to a namespace,while ClusterRole is not.

## Understanding the kubeconfig file

The kubeconfig file is used to configure access to Kubernetes clusters. It contains
the necessary information for kubectl to connect to the cluster, including the API
server address, user credentials, and context information.

The kubeconfig file is typically located at `~/.kube/config`, but you can specify
the custom location using the `KUBECONFIG` environment variable or the `--kubeconfig`.

We can view the current context using the following command:

```bash
kubectl config current-context
kubectl config view
```

If we lost the file, we can recreate it using the following command:

```bash
sudo cp /etc/kubernetes/admin.conf $HOME/.kube/config
```

> [!NOTE]
> The configuration file is stored in the `/etc/kubernetes` directory on the
> master node.

## Certificate Based Authentication

First we need to create a 509 certificate for the user,for this we can use the
openssl tool.We can use that certificate to authenticate to the kubernetes
cluster.

First need to generate a private key using the following command:

```bash
openssl genrsa -out kode.key 2048
```

Here,a new private key named `kode.key` is created with the size of 2048 bits.

We need to create a certificate signing request(CSR) for the information about
the certificate using the following command:

```bash
openssl req -new -key kode.key -out kode.csr -subj "/CN=kode/O=admin/OU=devops"
```

Then we need to sign the CSR using the Kubernetes CA to generate a client certificate
for the user using the following command:

```bash
openssl x509 -req -in ~/authentication/kode.csr \
  -CA /etc/kubernetes/pki/ca.crt \
  -CAkey /etc/kubernetes/pki/ca.key \
  -set_serial 101 \
  -extensions client \
  -days 365 \
  -outform PEM \
  -out ~/authentication/kode.crt
```

Now,we need to update or create kubeconfig file for the user.

```bash
kubectl config set-credentials kode \
  --client-certificate=~/authentication/kode.crt \
  --client-key=~/authentication/kode.key
```

This will create a new user named `kode` in the kubeconfig file with the
client certificate and key.

We can view using the following command:

```bash
kubectl config view
```

Now, We need to create a context for the user using the following command:

```bash
kubectl config set-context kode-context \
  --cluster=kubernetes \
  --user=kode \
  --namespace=default
```

Now,the new context is created for the user `kode` in the kubeconfig file.

We can switch to the new context using the following command:

```bash
kubectl config use-context kode-context
```

Lets try to list the pods in the default namespace using the following command:

```bash
kubectl get pods
```

### Role and RoleBinding

It is used to grant permissions within a specific namespace.

We will get forbidden error because the user `kode` does not have any permissions,
so we need to create a role and bind it to the user.

We can create role using the following yaml file:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pod-read-access
  namespace: default
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list", "watch", "create", "delete"]
```

And RoleBinding using the following yaml file:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-role-binding
  namespace: default
subjects:
  - kind: User
    name: kode
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-read-access
  apiGroup: rbac.authorization.k8s.io
```

> [!NOTE]
> Here [""] means the core API group.

### ClusterRole and ClusterRoleBinding

If we want to grant permissions across the entire cluster, we can use ClusterRole
and ClusterRoleBinding.

The following yaml file creates a ClusterRole that grants read access to pods.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: pod-read-access
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["get", "list"]
```

The cluster role binding binds the ClusterRole to the user `kode`.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pod-clusterrole-binding
subjects:
  - kind: User
    name: kode # Name is case sensitive
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-read-access
  apiGroup: rbac.authorization.k8s.io
```
