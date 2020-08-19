---
title: "apm.k8s.elastic.co/v1beta1"
linkTitle: ""
weight: 1
---

包装 v1beta1 包含为管理 APM 服务器资源的 API 模式定义 .

Resource Types: ApmServer

## ApmServer

ApmServer represents an APM Server resource in a Kubernetes cluster.

| Field      | Type          | Description                                                   |
| ---------- | ------------- | ------------------------------------------------------------- |
| apiVersion | string        | apm.k8s.elastic.co/v1beta1                                    |
| kind       | string        | ApmServer                                                     |
| metadata   | ObjectMeta    | Refer to Kubernetes API documentation for fields of metadata. |
| spec       | ApmServerSpec |                                                               |

## ApmServerSpec

ApmServerSpec holds the specification of an APM Server.

Appears In: ApmServer

| Field            | Type            | Description                                                                                                                             |
| ---------------- | --------------- | --------------------------------------------------------------------------------------------------------------------------------------- |
| version          | string          | Version of the APM Server.                                                                                                              |
| image            | string          | Image is the APM Server Docker image to deploy.                                                                                         |
| count            | integer         | Count of APM Server instances to deploy.                                                                                                |
| config           | Config          | Config holds the APM Server configuration. See: https://www.elastic.co/guide/en/apm/server/current/configuring-howto-apm-server.html    |
| http             | HTTPConfig      | HTTP holds the HTTP layer configuration for the APM Server resource.                                                                    |
| elasticsearchRef | ObjectSelector  | ElasticsearchRef is a reference to the output Elasticsearch cluster running in the same Kubernetes cluster.                             |
| podTemplate      | PodTemplateSpec | PodTemplate provides customisation options (labels, annotations, affinity rules, resource requests, and so on) for the APM Server pods. |
| secureSettings   | SecretSource    | SecureSettings is a list of references to Kubernetes secrets containing sensitive configuration options for APM Server.                 |
