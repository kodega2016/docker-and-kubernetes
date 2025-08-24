# Authentication and Authorization in Kubernetes

<!--toc:start-->

- [Authentication and Authorization in Kubernetes](#authentication-and-authorization-in-kubernetes)
  - [Introduction](#introduction)
    - [Authentication Methods](#authentication-methods)
    - [Authorization Methods](#authorization-methods)
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
