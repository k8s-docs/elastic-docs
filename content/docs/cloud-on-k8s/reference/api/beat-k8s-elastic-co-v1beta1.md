---
title: "beat.k8s.elastic.co/v1beta1"
linkTitle: ""
weight: 1
---

Package v1beta1 contains API Schema definitions for the beat v1beta1 API group

Resource Types

Beat

## Beat

Beat is the Schema for the Beats API.

| Field      | Type       | Description                                                   |
| ---------- | ---------- | ------------------------------------------------------------- |
| apiVersion | string     | beat.k8s.elastic.co/v1beta1                                   |
| kind       | string     | Beat                                                          |
| metadata   | ObjectMeta | Refer to Kubernetes API documentation for fields of metadata. |
| spec       | BeatSpec   |                                                               |

## BeatSpec

BeatSpec defines the desired state of a Beat.

Appears In: Beat

| Field              | Type           | Description                                                                                                                                                                                                                                                                                                                                 |
| ------------------ | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| type               | string         | Type is the type of the Beat to deploy (filebeat, metricbeat, heartbeat, auditbeat, journalbeat, packetbeat, etc.). Any string can be used, but well-known types will have the image field defaulted and have the appropriate Elasticsearch roles created automatically. It also allows for dashboard setup when combined with a KibanaRef. |
| version            | string         | Version of the Beat.                                                                                                                                                                                                                                                                                                                        |
| elasticsearchRef   | ObjectSelector | ElasticsearchRef is a reference to an Elasticsearch cluster running in the same Kubernetes cluster.                                                                                                                                                                                                                                         |
| kibanaRef          | ObjectSelector | KibanaRef is a reference to a Kibana instance running in the same Kubernetes cluster. It allows automatic setup of dashboards and visualizations.                                                                                                                                                                                           |
| image              | string         | Image is the Beat Docker image to deploy. Version and Type have to match the Beat in the image.                                                                                                                                                                                                                                             |
| config             | Config         | Config holds the Beat configuration. At most one of [Config, ConfigRef] can be specified.                                                                                                                                                                                                                                                   |
| configRef          | ConfigSource   | ConfigRef contains a reference to an existing Kubernetes Secret holding the Beat configuration. Beat settings must be specified as yaml, under a single "beat.yml" entry. At most one of [Config, ConfigRef] can be specified.                                                                                                              |
| secureSettings     | SecretSource   | SecureSettings is a list of references to Kubernetes Secrets containing sensitive configuration options for the Beat. Secrets data can be then referenced in the Beat config using the Secret’s keys or as specified in Entries field of each SecureSetting.                                                                                |
| serviceAccountName | string         | ServiceAccountName is used to check access from the current resource to Elasticsearch resource in a different namespace. Can only be used if ECK is enforcing RBAC on references.                                                                                                                                                           |
| daemonSet          | DaemonSetSpec  | DaemonSet specifies the Beat should be deployed as a DaemonSet, and allows providing its spec. Cannot be used along with deployment. If both are absent a default for the Type is used.                                                                                                                                                     |
| deployment         | DeploymentSpec | Deployment specifies the Beat should be deployed as a Deployment, and allows providing its spec. Cannot be used along with daemonSet. If both are absent a default for the Type is used.                                                                                                                                                    |

## DaemonSetSpec

Appears In: BeatSpec

| Field       | Description     |
| ----------- | --------------- |
| podTemplate | PodTemplateSpec |

## DeploymentSpec

Appears In: BeatSpec

| Field       | Description     |
| ----------- | --------------- |
| podTemplate | PodTemplateSpec |
| replicas    | integer         |
