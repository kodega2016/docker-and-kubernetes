# Namespace

<!--toc:start-->

- [Namespace](#namespace)
  - [Introduction](#introduction)
  <!--toc:end-->

## Introduction

We can use namespace in kubernetes to group the resources,We can create
resources on that particular group, assigned roles based on that groups.

To create the namespace we can run the following command.

```bash
kubectl create namespace <namespace-name>
```

We also can create namespace using the yaml file.

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: dev-lab
  labels:
    author: kodega
```
