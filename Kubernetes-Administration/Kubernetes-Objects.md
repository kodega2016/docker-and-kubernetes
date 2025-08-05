# Kubernetes Objects Overview

<!--toc:start-->

- [Kubernetes Objects Overview](#kubernetes-objects-overview)
  - [What is a Pod?](#what-is-a-pod)
  - [What is initContainers?](#what-is-initcontainers)
  - [POD LifeCycle Restart Policy](#pod-lifecycle-restart-policy)
  - [Delete Cascade](#delete-cascade)
  - [Replication Controller](#replication-controller)
  - [ReplicaSet](#replicaset)
  - [Deployment](#deployment)
  - [DaemonSet](#daemonset)
  <!--toc:end-->

In Kubernetes,each of the components is represented as an object.We
need to create these objects to run our applications in the kubernetes
cluster.

Workload objects:

- pods
- replicaset or replication controller
- deployments
- daemonsets
- statefulsets
- jobs or cronjobs

Some of the other objects are:

- services
- configmaps
- secrets
- persistent volumes
- persistent volume claims
- network policies
- namespaces
- ingress

## What is a Pod?

It is one of the smallest unit in the kubernetes,we can run multiple
containers in pod,normally related containers are running on the single
pod,they are also known as side car containers.

The pod is the logical unit for the shared volumes and shared networking
between containers,they can communicate with using the localhost.

We can have resources limits and requests for the containers.The
containers can have memory and cpu requests and limits.

## What is initContainers?

They are the special containers,they runs before the main containers
starts.They are used for the initialization tasks such as setting up
scripts,Each initContainers starts and finish and others containers starts.

initContainers can have their own images and configurations.

```yaml
initContainers:
  - name: alpine-init
    image: alpine
    command:
      - wget
      - "-O"
      - "/var/tmp/index.html"
      - https://github.com/sudheerduba/initContainer_demo/blob/main/index.html?raw=true
    volumeMounts:
      - name: shared-volume
        mountPath: /var/tmp
```

## POD LifeCycle Restart Policy

We can have following restart policies for the pods:

- Always: Pod will be restarted if it fails.
- Never: Pod will not be restarted if it fails.
- OnFailure: Pod will be restarted if it fails, but not if it is terminated by
  the exit code 0.

We can scale the replication controller using the following command:

```bash
kubectl scale --replicas 2 rc/replication-controller-name
```

## Delete Cascade

When we delete the replication controller,by default it will delete the pods
but we can also delete the parent controller using the `--cascade` option.

```bash
kubectl delete --cascade=orphan rc nginx-rc
```

## Replication Controller

It is used to ensure that a specified number of pod replicas are running at any
given time.This is useful for scaling applications and ensuring high availability.

```yaml
apiVersion: v1
kind: ReplicationController
metadata:
  name: nginx-rc
spec:
  replicas: 3
  selector:
    app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:latest
          ports:
            - containerPort: 80
```

```bash
kubectl get rc
```

## ReplicaSet

It is a newer version of the replication controller and is used to ensure that a
number of pod replicas are running at any given time,similar to the replication
controller.

It is used in deployments to manage the pods and ensure that the desired number
of replicas are running.

The api version for the replicaset is `apps/v1` and the kind is `ReplicaSet`.

It is recommended to use Deployment instead of ReplicaSet directly,as the
deployment provides additional features such as rolling updates and
rollbacks.

## Deployment

Like the replication controller and replicaset, it is used to manage the number
of pod replicas and ensure that the desired number of replicas are running.

It has features such as rolling updates and rollbacks,which makes it easier to
roll out changes to the application without downtime.

It is the most commonly used object in Kubernetes for managing the pods and
deployments.

We

```bash
kubectl set image deployment/nginx-deployment nginx=nginx:1.19.0
```

To roll back to previous version:

```bash
kubectl rollout undo deployment/nginx-deployment --to-revision=1
```

To check the rollout status:

```bash
kubectl rollout status deployment/nginx-deployment
```

By default,deployment follows the `RollingUpdate` strategy,which means that it will
only update a few pods at a time,ensuring that the application remains available
during the update process.

There are two update strategies available for deployments:

- `RollingUpdate`: This is the default strategy and it updates the pods in a rolling
  manner,ensuring that the application remains available during the update process.

- `Recreate`: This strategy will terminate all the existing pods and create new ones,
  which may cause downtime for the application.

There are two key properties in the deployment process maxSurge and maxUnavailable:

MaxSurge: This is the maximum number of pods that can be created above the desired
number of pods during the update process.

MaxUnavailable: This is the maximum number of pods that can be unavailable during
the update process.

## DaemonSet

DaemonSet is used to ensure that at least one pod is running on each node in the
cluster.It is useful for running background tasks such as logging,monitoring and
other system-level tasks.
