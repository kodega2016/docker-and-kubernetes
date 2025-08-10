# Services in Kubernetes

<!--toc:start-->

- [Services in Kubernetes](#services-in-kubernetes)
  - [What is Service?](#what-is-service)
  - [ClusterIP Service](#clusterip-service)
  - [NodePort Service](#nodeport-service)
  - [Customizing NodePort Service IP ranges](#customizing-nodeport-service-ip-ranges)
  - [LoadBalancer Service](#loadbalancer-service)
  - [MetalLB for On-Premises Load Balancing](#metallb-for-on-premises-load-balancing)
  - [ExternalIP Service](#externalip-service)
  - [ExternalName Service](#externalname-service)
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

We also can run the following command to expose a service using the NodePort type:

```bash
kubectl expose deployment <deployment-name> --type=NodePort \n
--name=<service-name> \n
--port=<port> --target-port=<target-port> \n
--selector=<label-selector>
```

## Customizing NodePort Service IP ranges

The kubernetes configuration file is located at `/etc/kubectnetes` folder.So
we also can customize the default NodePort service IP ranges by editing the

There we can customize the default NodePort service IP ranges by editing the
`/etc/kubernetes/manifests/kube-apiserver.yaml` file and adding the
`--service-node-port-range` flag to the `kube-apiserver` container.

```yaml
--service-node-port-range=30000-32767
```

## LoadBalancer Service

It creates an external load balancer in a cloud provider and assigns a fixed,
address to the Service. It is used for exposing Services to the internet.

While using the cloud provider,it uses the cloud provider's load balancer
service to route traffic to the Service.

On-premises,we can use different load balancer solutions like MetalLB,
OpenELB or PureLB to achieve the same functionality.

For the cloud,there are different load balancer solutions like AWS ELB, GCP
Load balancer,

AWS- AWS Net work load balancer
AZURE- Azure Load Balancer
GCP- GCP Load Balancer

## MetalLB for On-Premises Load Balancing

We can use MetalLB to provide load balancing for services in an on-premises
server. MetalLB is a load balancer implementation for bare metal Kubernetes
clusters. It allows you to expose services using a LoadBalancer type service.

Components of MetalLB:

- **Controller**: Runs in the Kubernetes cluster and manages the
  allocation of IP addresses for LoadBalancer services.It is deployed
  as `deployment`

- **Speaker**: Runs on each node and announces the IP addresses
  allocated by the controller to the network.It is deployed as a `DaemonSet`.

> [!NOTE]
> The kube-proxy has three modes: `iptables`, `ipvs`, and `userspace`.We
> must have `ipvs` mode enabled to use MetalLB.

## ExternalIP Service

It is a type of Service that allows you to expose a Service using an
ip address that is already assigned to a node in the cluster.

“I already have an IP that clients can hit — just forward traffic from it to my Pods.”

## ExternalName Service

ExternalName is a special type of Service that maps a Service to a DNS name,It
will not create any cluster ip or load balancer,it simply maps the CNAME to
an external DNS name.

For example,if we want to map a database service url to external name, we can use
ExternalName service,so that we can access the database service using the cluster
ip service name.

It helps to configure CNAME records in DNS servers,so that we can access the
external service using the cluster ip service name.

We can use the following url to access the external service:

```bash
ext-name-svc.default.svc.cluster.local
```
