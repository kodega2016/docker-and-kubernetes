# Authentication and Authorization in Kubernetes

<!--toc:start-->

- [Authentication and Authorization in Kubernetes](#authentication-and-authorization-in-kubernetes)
  - [Introduction](#introduction)
    - [Authentication Methods](#authentication-methods)
    - [Authorization Methods](#authorization-methods)
  - [Kubernetes Authentication Strategies](#kubernetes-authentication-strategies)
  - [Kubernetes Authorization Strategies](#kubernetes-authorization-strategies) - [Role-Based Access Control (RBAC)](#role-based-access-control-rbac)
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
> Role is scoped to a namespace,while ClusterRole is not
Role can be defined using the following synatx.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: dev
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

Role Binding can be defined using the follwoing synax:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: read-pods-binding
  namespace: dev
subjects:
- kind: User
  name: alice        # this user
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```
