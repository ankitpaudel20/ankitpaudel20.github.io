---
title: "Cloud-native monitoring with Lightweight Prometheus + Thanos on GKE"
description: How to setup lightweight HA Prometheus + Thanos cluster for verbose monitoring on gke clusters with more than 50% cost-savings as compared to google managed prometheus service.
date: 2025-08-10T08:33:09Z
image: "grafana-thanos.png"
math: 
license: 
hidden: false
comments: true
draft: false
categories:
  - kubernetes
tags:
  - kubernetes
  - monitoring
---

## Preface

* You require a cheap monitoring solution that is cloud agnostic.

## Problem

* Google's manged prometheus costs a lot; so self hosting prometheus+thanos to get around it.
  * As the required metrics to properly observe our application increases, the requirement to selfhost the monitoring platform to curb the cloud costs makes more sense, rather than using managed monitoring service.
  * This opens up self hosting and maintaining Prometheus+Thanos+GCS stack for all the internal observability needs.
  * low cost of GCS + infrequent access to older metrics data makes **Thanos**'s requirement more sensible.

## The Solution

* Store less data in prometheus's tsdb while maintaining historical data in gcs for low cost query.
  * Setup proper retention period as business requirement to reduce the latency of metrics query.

## Prometheus

* A stable and mature TSDB, designed to record and host application metrics.
* **Pros:**
  * Easy to get started with grafana for basic monitoring needs.
  * Low barrier to entry if the requirement is not high availability.
* **Cons:**
  * Hard to scale if HA is explicity required.
  * Multiple Prometheus instances work independently, it does not have built in feature for master-slave replication and HA deployment like postgres and other stateful applications.

## Thanos

* Solution built on top of Prometheus to solve HA and scalibility needs.
* Requires active connection to Prometheus instance to function.
* Manages multiple Prometheus instances and handles deduplication, downsampling and other higher order features that Prometheus does not support natively.

## High level proposed architecture

* Store as less of data as required in memory in the prometheus instances.
* Configure thanos side car to upload metric data as frequent as possible to gcs or other colder storage as compared to tsdb for low memory and cluster resource requirement.
  
### Possible issues with this approach

* Chatty communication between monitoring components might incur higher networking bills for multi-region k8s clusters.
* Needs manual security considerations to secure grafana and other public endpoints.
* Might take up engineering bandwidth: requiring more engineers to maintain it in the long term.

## Installation

* This uses helm-charts by bitnami for thanos and the official Prometheus community maintained helm chart.
* Optimized values for helm install of our specific use case:
  * Chart used: [kube_prometheus_stack](https://github.com/prometheus-community/helm-charts) and [bitnami_thanos_chart](https://artifacthub.io/packages/helm/bitnami/thanos).
  * Image tags are latest as per the period of this article.
* **Notes** 
  * Some names like the service endpoints might be changed as per the updation of helm chart and your release names.
  * This is before bitnami announced its paid offerings.
    
```yaml
# Prometheus
alertmanager: # not used, in our case.
  enabled: false
coreDns:
  enabled: false
grafana:
  datasources:
    datasources.yaml:
      apiVersion: 1
      datasources:
      - access: proxy
        isDefault: false
        jsonData:
          prometheusType: thanos
        name: prom-default
        type: prometheus
        url: http://thanos-query-frontend.monitoring.svc.cluster.local:9090
  deploymentStrategy:
    type: Recreate # we are using RWO persistence
  persistence:
    enabled: true
    size: 5Gi
kube-state-metrics:
  enabled: true
kubeApiServer:
  enabled: false
kubeControllerManager:
  enabled: false
kubeProxy:
  enabled: false
kubeScheduler:
  enabled: true
kubeStateMetrics:
  enabled: true
kubelet:
  enabled: true
nodeExporter:
  enabled: false
prometheus:
  prometheusSpec:
    containers:
    - env:
      - name: GOGC
        value: "75"
      name: prometheus
    externalLabels:
      cluster: <label-identifying-your-cluster-for-multicluster-setups>
      replica: $(POD_NAME)
    podMonitorSelectorNilUsesHelmValues: false
    replicas: 2
    retention: 3h
    retentionSize: 8GiB
    serviceMonitorSelectorNilUsesHelmValues: false
    storageSpec:
      volumeClaimTemplate:
        spec:
          accessModes:
          - ReadWriteOnce
          resources:
            requests:
              storage: 10Gi
          storageClassName: premium-rwo # this is specific to GCP, change as you require
    thanos:
      objectStorageConfig:
        existingSecret: # this is to access gcs for metrics storage; for aws, use per documentation of thanos.
          key: objstore.yml
          name: thanos-config
  thanosService:
    enabled: true
prometheus-node-exporter:
  enabled: false # This is covered by the provided metrics of gke.
prometheusOperator:
  admissionWebhooks:
    patch:
      image:
        tag: v1.5.2 
  thanosImage:
    tag: v0.39.2
```

``` yaml
# Thanos
bucket:
  enabled: true
  replicaCount: 1
compactor:
  cronJob:
    activeDeadlineSeconds: 10800
    enabled: true
    failedJobsHistoryLimit: 3
    schedule: 0 */6 * * *
    successfulJobsHistoryLimit: 2
  enabled: true
  extraFlags:
  - --delete-delay=12h
  - --deduplication.replica-label=replica
  - --deduplication.replica-label=prometheus_replica
  persistence:
    enabled: false
  retentionResolution1h: 0d
  retentionResolution5m: 180d
  retentionResolutionRaw: 30d
existingObjstoreSecret: thanos-config
existingServiceMonitor: false
metrics:
  enabled: true
  serviceMonitor:
    enabled: true
objectStorageConfig:
  existingSecret:
    key: objstore.yml
    name: thanos-config
query:
  dnsDiscovery:
    sidecarsNamespace: monitoring
    sidecarsService: prometheus-stack-kube-prom-thanos-discovery
  enabled: true
  extraFlags:
  - --query.auto-downsampling
  - --query.partial-response
  - --query.timeout=2m
  replicaCount: 2
queryFrontend:
  enabled: true
  extraFlags:
  - --query-range.split-interval=24h
  - --query-range.max-retries-per-request=3
  - --query-frontend.compress-responses
  - --query-range.partial-response
  replicaCount: 2
ruler:
  enabled: false
storegateway:
  enabled: true
  extraFlags:
  - --index-cache-size=512MB
  - --chunk-pool-size=512MB
  - --store.grpc.series-max-concurrency=10
  - --block-sync-concurrency=3
  - --store.enable-index-header-lazy-reader
  persistence:
    enabled: true
    size: 5Gi
    storageClass: premium-rwo
  replicaCount: 2
```

## Final notes and recommendations
* Setup proper alerting for failed compactors and other important metrics like consistent storage-gateway cache misses.
  * Use proper memory requirements for storage gateway and querier for faster queries as business requirements.
* This setup does not factor in the cardinality constraints of your specific usecase.
  * You might be a startup just starting with low enough unique users or a B2B product with low enough user-base that can get away with taking in user-id as a label in prom-metrics, but this might blow up your prom instance if your userbase is large enough.
* Keep monitoring the resource usage and latency of thanos and prom components for few days/weeks for resource exhaustion and your needs. **If your prom starts getting OOM failures, you might need to reconfigure this setup to use remote-write instead of sidecar based pattern.**
  * Remote-Write is a little complicated but is a good trade-off if you have a quite a few components that queries for latest metrics and have enough engineering bandwidth to maintain it in the long term.
* Use a separate Node pool for monitoring components like prometheus and stateful components like databases and distributed caches for resource isolation and to reduce failure domains.