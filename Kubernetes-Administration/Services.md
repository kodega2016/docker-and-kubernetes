# Services in Kubernetes

<!--toc:start-->

- [Services in Kubernetes](#services-in-kubernetes)
  - [What is Service?](#what-is-service)
  - [ClusterIP Service](#clusterip-service)
  - [NodePort Service](#nodeport-service)
  <!--toc:end-->

We are going to cover the following topics in this section:

- What is a Service?
- Types of Services
- How to create a Service
- How to access a Service

## What is Service?

It is a logical abstraction that defines a set of Pods and a policy by which to
access them. Services enable communication between different components in a
Kubernetes cluster, such as between Pods, or between Pods and external clients.

There are different types of Services in Kubernetes, each serving a specific
purpose:

- **ClusterIP**: The default type, which exposes the Service on a cluster-
  internal IP.
  It is only accessible within the cluster.

- **NodePort**: Exposes the Service on each Node's IP at a static port.
  It allows external access to the Service by routing traffic to the
  Node's IP and port.

- **LoadBalancer**: Creates an external load balancer in a cloud provider and
  assigns a fixed, external IP to the Service. It is used for exposing
  Services to the internet.

- **ExternalName**: Maps the Service to a DNS name, allowing access to an
  external resource

Each service will have a unique IP address and DNS name within the cluster,
the DNS name will be assigned by using the CoreDNS service.

## ClusterIP Service

It is the default type of Service in Kubernetes. It exposes the Service on a
cluster-internal IP address, making it accessible only within the cluster.

The kube-proxy component is responsible for routing traffic to the appropriate
pods based on the Service's selector.

The dns name looks like this:

```
<service-name>.<namespace>.svc.cluster.local
```

We can also view all the endpoints of a service using the following command:

```bash
kubectl get ep <service-name>
```

we also can run the following command to expose a service to the cluster:

```bash
kubectl expose deployment <deployment-name> --type=ClusterIP \n
--name=<service-name> \n
--port=<port> --target-port=<target-port> \n
selector=<label-selector>
```

## NodePort Service

It exposes the servics on each node's IP at a static port,when traffic comes
to the nodes IP and port, it is forwared to the service and then to the pods.

The port range for NodePort service is `30000-32767`.

The workflow looks like this:
external client-> Node IP:NodePort-> Service -> Pod IP:TargetPort

> [!NOTE]
> We need to allow the request to the NodePort if the firewall is enabled
> on the nodes.
