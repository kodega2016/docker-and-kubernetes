# Stateful Sets

<!--toc:start-->

- [Stateful Sets](#stateful-sets)
  - [Introduction](#introduction)
  <!--toc:end-->

## Introduction

Stateful Sets are workloads in kubernetes that manage stateful applications.
It manages pods with stable network identities and persistent storage.

It is used to run the application such as databases, message queues, and other
stateful applications that require stable network identities and persistent storage.

The pods get static hostname and stable network identity.The pods are created in
order and are terminated in reverse order.

Each of the pods has its own persistent storage, which is created and managed by
the Stateful Set controller.

Each of the pods get persistent dns name and the pods name also follows the
same.

`<sts-name>-<index>.<headless-service-name>.<namespace>.svc.cluster.local`

> [!NOTE]
> We must use a headless service to access the pods in a Stateful Set.
