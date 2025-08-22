# Scheduling Workloads

<!--toc:start-->

- [Scheduling Workloads](#scheduling-workloads)
  - [Introduction to Scheduling Workloads](#introduction-to-scheduling-workloads)
  - [Scheduler Workloads on Kubernetes](#scheduler-workloads-on-kubernetes) - [nodeName](#nodename)
  <!--toc:end-->

## Introduction to Scheduling Workloads

In this section, we will explore how to schedule workloads in Kubernetes.
Scheduling is the process of assigning pods to nodes in a cluster based
on resource availability and constraints.

Steps:

**_Kubectl_**
Developers or admin run `kubectl` command to create workloads,
then the command is sent to the kubernetes API server.

It will first do verify the authentication and authorization to validate
the requester has the permission to create the workload.

After that,the API server will validate the schema of the request.

Once it is validated, It will send the request to the etcd,etcd will check if the
requested resource is already exist,or not, if not, it will store the request in
the etcd.

And return the response to the API server, then the API server will return the response
to the requester.

Once the response is got by the API server, it will send the request to the scheduler
to schedule the workload.The scheduler will check the available nodes and send the
acknowledgement to the API server.

After that,The API server will send the request to one of the available nodes,where
the kubelet is running.Then the kubelet will create the resource on the node.
Finally, the kubelet will send the response to the API server, and the API server
will be responsible for updating the status of the resource in the etcd.

And the final step is the API server will send the response to the requester to
the client.

## Scheduler Workloads on Kubernetes

In control plane,the scheduler will watch for the newly created pods which are not
assigned to any node.

Scheduler takes the responsibility of assigning the pods to the appropriate nodes(compute
plane node mostly) based on the resource availability and constraints.

Feasible nodes are the nodes where the pods can be scheduled based on the resources
and constraints.

kube scheduler updated the api server once decided the node for the pod whose information
will be stored in the etcd.

The major steps of scheduling are:

- Filtering: The scheduler filters out the nodes that do not meet the
  requirements of the pod.
- Scoring: The scheduler scores the remaining nodes based on various factors
  such as resource availability,

Methods to schedule workloads:

- nodeName
- nodeSelector
- nodeAffinity
- podAffinity
- taints and tolerations

### nodeName

We can directly assign a pod to a specific node by specifying the nodeName field
in the pod specification.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-pod
spec:
  nodeName: node-1
  containers:
    - name: my-container
      image: my-image
```

> [!NOTE]
> We can pass only one node name in the nodeName field.
