# Kubernetes Monitoring

<!--toc:start-->

- [Kubernetes Monitoring](#kubernetes-monitoring)
  - [Introduction](#introduction)
  <!--toc:end-->

## Introduction

We need tools to monitor the health and performance of the kubernetes cluster
and applications running on it. Monitoring helps in identifying issues,bottlenecks,
and ensuring the overall reliability of the system.

What we should monitor in a kubernetes cluster?

- Cluster health and performance
- Node metrics (CPU, memory, disk usage)
- Ensure all the pods are running as expected
- Application performance metrics

Also trouble shooting issues in a kubernetes cluster can be challenging.

- Node Level logs capture the logs of the node and its components.
- Pod Level logs capture the logs of the application running inside the pod.
- kuberenetes events and audit logs help in identifying issues related to
  resource creation, deletion, and modification.

Open source tools for monitoring kubernetes:

- Prometheus
- Grafana
- ELK Stack (Elasticsearch, Logstash, Kibana)
