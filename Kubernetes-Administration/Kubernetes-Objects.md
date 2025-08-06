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
  - [Jobs](#jobs)
  - [CronJobs](#cronjobs)
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

It adds a pod to each node in the cluster,ensuring that the pod is running on
each node.

UseCases:

- running a logging agent on each node of the cluster.
- running a monitoring agent on each node of the cluster.

There are OnDelete strategy which means that the pods will not be updated automatically,
we need to delete the pods manually to update them.

## Jobs

They are used to run a batch jobs or a one-time tasks in the cluster such as data
processing,backup and restore tasks.

It ensures that one o more pods are running and terminated after the job is completed.

The api version for the job is `batch/v1` and the kind is `Job`.

The restart policy for the job is either `Never` or `OnFailure` which means
that the job will not run always and will not be restarted if it fails.

The restartPolicy Never means that the job will not be restarted if it fails,
It will starts a new pod if it fails and will not restart the existing pod.

But when we use the restartPolicy OnFailure, it will restart the pod if it fails
to ensure that the job is completed successfully.

We can also set `ttlSecondsAfterFinished` to specify the time to live for the job
after it is completed.

## CronJobs

They are used to run a batch jobs at a specific time or interval,similar to
the cron jobs in Linux.

The cronjob will starts a jobs at a specific time or interval and will
ensure that the job is completed successfully.

The timezone will be the timezone of the kubecontrol plane.We also
can set the timezone for the cronjob using the `spec.timezone` field.

We can set concurrencyPolicy to control how many jobs can run at the same time.
The options are:

- `Allow`: allows multiple jobs to run at the same time.
- `Forbid`: prevents multiple jobs from running at the same time.
- `Replace`: replaces the currently running job with a new one.
