# Config Maps and Secrets

<!--toc:start-->

- [Config Maps and Secrets](#config-maps-and-secrets)
  - [Introduction](#introduction)
  - [ConfigMaps](#configmaps)
  <!--toc:end-->

## Introduction

We will explore how to use ConfigMaps and Secrets in Kubernetes to manage
configuration data and sensitive information.

We use config maps to store non-sensitive configuration data that can be
either in a file or in a key-value pair format. Secrets are used to store
the sensitive information such as passwords, tokens, and SSH keys.

## ConfigMaps

We will store the non sensitive configuration data in a ConfigMap.It is a
key-value store that can be used to store configuration data.

The maxium size of a ConfigMap is 1MB. The data can be stored in a file or in a
key-value pair format.

The environment variables can be used by the pods.

We can create config maps using both the imperative and declarative
approach.

```# Imperative approach
kubectl create configmap appconfig --from-literal=key1=value1 --from-literal=key2=value2
```

We can list the config maps using the following command:

```bash
kubectl get configmaps
kubectl get configmap appconfig -o yaml
```

For example.

```bash
kubectl create configmap mysqlconfig \
  --from-literal=MYSQL_ROOT_PASSWORD=supersecret \
  --from-literal=MYSQL_USER=kodega \
  --from-literal=MYSQL_PASSWORD=thekeythekey \
  --from-literal=MYSQL_DATABASE=lovethecode
```


We can also create configmap using the file.
```bash
kubectl create configmaps mysqlconfig \
  --from-file=dbconfig.properties.
```