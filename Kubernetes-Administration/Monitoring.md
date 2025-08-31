# Kubernetes Monitoring

<!--toc:start-->

- [Kubernetes Monitoring](#kubernetes-monitoring)
  - [Introduction](#introduction)
  - [Prometheus](#prometheus)
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
  or EFK Stack (Elasticsearch, Fluentd, Kibana)

## Prometheus

It os open source monitoring and alerting tools designed for reliability and scalability.It
collects metrics from various sources including kubernetes clusters, applications,
and services and setup alerts based on predefined thresholds.

It stores the time series data in key-value pairs and provides a powerful query language
to retrieve and analyze the data.

Prometheus architecture:

- Prometheus server: It is the core component that collects and stores the metrics
  data.

- Exporters: These are the components that expose the metrics data in a format that
  the Prometheus server can scrape. There are various exporters available for different
  platforms and applications.

- Alertmanager: It is responsible for managing and sending alerts based on the
  predefined rules.

- Pushgateway: It is used to push the metrics data from short-lived jobs or batch
  processes.

- PromQL: It is a powerful query language used to retrieve and analyze the metrics
  data stored in Prometheus.

- Grafana: It is a popular open source tool for visualizing the metrics data collected
  by Prometheus.
