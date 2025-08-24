# Scheduling Workloads

<!--toc:start-->

- [Scheduling Workloads](#scheduling-workloads)
  - [Introduction to Scheduling Workloads](#introduction-to-scheduling-workloads)
  - [Scheduler Workloads on Kubernetes](#scheduler-workloads-on-kubernetes) - [nodeName](#nodename) - [nodeSelector(exact match)](#nodeselectorexact-match) - [nodeAffinity operators](#nodeaffinity-operators) - [Rules in nodeAffinity](#rules-in-nodeaffinity) - [Pod Affinity](#pod-affinity) - [Taints and Tolerations](#taints-and-tolerations) - [Anatomy of a Taint](#anatomy-of-a-taint)
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

### nodeSelector(exact match)

We can use nodeSelector to schedule pods based on labels assigned to nodes.
To assign the labels to the nodes, we can use the following command:

```bash
kubectl label nodes <node-name> <label-key>=<label-value>
```

To remove a label from a node, we can use the following command:

```bash
kubectl label nodes <node-name> <label-key>-
```

Then we can pass the same label in the pod specification to schedule the pod
to the node with the matching label.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  nodeSelector:
    disktype: ssd
  containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
```

> [!NOTE]
> It support exact match for the key-value pair.If the node with the matching
> key-value pair is not found, the pod will remain in the pending state.

### nodeAffinity operators

We can use different operators with nodeAffinity to schedule pods based on labels
using different matching criteria.

**_In_**:
It checks if the node's label value is in the specified list of values.

```yaml
- key: disktype
  operator: In
  values:
    - ssd
    - hdd
```

It must have at least one value in the values list.

**_NotIn_**:
It checks if the node's label value is not in the specified list of values.

```yaml
- key: disktype
  operator: NotIn
  values:
    - ssd
```

**_Exists_**:

- It checks if the node has the specified label key, regardless of its value.

```yaml
- key: disktype
  operator: Exists
```

**_DoesNotExist_**:
It checks if the node does not have the specified label key.

```yaml
- key: disktype
  operator: DoesNotExist
```

GT (Greater Than) and LT (Less Than):
We also have the option to use GT and LT operators to compare numeric values.

### Rules in nodeAffinity

There are two types of rules in nodeAffinity:

- RequiredDuringSchedulingIgnoredDuringExecution
- PreferredDuringSchedulingIgnoredDuringExecution

RequiredDuringSchedulingIgnoredDuringExecution:
It is a hard requirement, which means that the pod will only be scheduled
after finding a node that meets the criteria.

It is required during scheduling but ignored during execution, meaning that
the pod will not be evicted if the node's labels change after the pod is scheduled.

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
      - matchExpressions:
          - key: env
            operator: In
            values:
              - dev
              - prod
```

> [!NOTE]
> Here, the pods will be scheduled only on the nodes,which have the label
> either dev or prod.

We can remove the label from the resources using the following command.

```bash
kubectl label node k8s-worker1 env-
```

> [!NOTE]
> After the pod is scheduled, if we remove the label from the node,it will not
> remove the pod from the node.

### Pod Affinity

It is used to schedule pods based on the labels of other pods.so if we want to
run a pod on the same node as another pod, we can use pod affinity.

It is useful for co-locating pods that need to communicate with each other
such as application and its database.

It also has two types of rules:

- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution

We need to define the topologyKey also for the requiredDuringSchedulingIgnoredDuringExecution
rules:

```yaml
topologyKey: kubernetes.io/hostname
```

We also can apply the anti-affinity with this.

```yaml
affinity:
  podAffinity:
    preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
              - key: app
                operator: In
                values:
                  - web
          topologyKey: "kubernetes.io/hostname"
```

### Taints and Tolerations

In production,we want to control which pods can be scheduled on which nodes.For example,
we may want to run specific workloads such as gpu workloads on specific nodes.

Some nodes should not run any workloads, such as control plane nodes.

Taints marks the nodes to repel certain pods from being scheduled on them whereas
tolerations allow pods to be scheduled on nodes with specific taints.

#### Anatomy of a Taint

A taint has three parts:

- key
- value
- effect

Here,key and value are arbitrary strings that we define, and effect can be one of
the following:

- NoSchedule: Pods that do not tolerate this taint will not be scheduled on the node.
- PreferNoSchedule: The scheduler will try to avoid placing pods that do not tolerate
  the taint on the node, but it is not guaranteed.
- NoExecute: Pods that do not tolerate this taint will be evicted from the node

```bash
kubectl taint nodes worker-1 gpu=true:NoSchedule
```

So now, any pod that does not have a toleration for this taint will not be scheduled,
to run a pod on this node, we need to add the following toleration in the pod specification.

```yaml
tolerations:
  - key: "type"
    operator: "Equal"
    value: "master"
    effect: "NoSchedule"
```

To set the tains for the master node, we can use the following command:

```bash
kubectl taint nodes k8s-master type=master:NoSchedule
```

To allow the pods to be scheduled on the master node, we need to add the above toleration
in the pod specification.

```yaml
tolerations:
  - key: "type"
    operator: "Equal"
    value: "master"
    effect: "NoSchedule"
```
