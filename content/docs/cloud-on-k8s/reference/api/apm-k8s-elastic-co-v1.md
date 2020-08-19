---
title: "apm.k8s.elastic.co/v1"
linkTitle: ""
weight: 1
---

包 V1 包含 API 模式定义 管理 APM 服务器资源.

- 资源类型

  ApmServer

## ApmServer

ApmServer 代表 Kubernetes 集群里 APM 服务器资源 .

| Field      | Type          | Description                                                   |
| ---------- | ------------- | ------------------------------------------------------------- |
| apiVersion | string        | apm.k8s.elastic.co/v1                                         |
| kind       | string        | ApmServer                                                     |
| metadata   | ObjectMeta    | Refer to Kubernetes API documentation for fields of metadata. |
| spec       | ApmServerSpec |                                                               |

## ApmServerSpec

ApmServerSpec holds the specification of an APM Server.

Appears In: ApmServer

| Field              | Type            | Description                                                                                                                                           |
| ------------------ | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------- |
| version            | string          | Version of the APM Server.                                                                                                                            |
| image              | string          | Image is the APM Server Docker image to deploy.                                                                                                       |
| count              | integer         | Count of APM Server instances to deploy.                                                                                                              |
| config             | Config          | Config holds the APM Server configuration. See: [1]                                                                                                   |
| http               | HTTPConfig      | HTTP holds the HTTP layer configuration for the APM Server resource.                                                                                  |
| elasticsearchRef   | ObjectSelector  | ElasticsearchRef is a reference to the output Elasticsearch cluster running in the same Kubernetes cluster.                                           |
| kibanaRef          | ObjectSelector  | KibanaRef is a reference to a Kibana instance running in the same Kubernetes cluster. It allows APM agent central configuration management in Kibana. |
| podTemplate        | PodTemplateSpec | PodTemplate provides customisation options (labels, annotations, affinity rules, resource requests, and so on) for the APM Server pods.               |
| secureSettings     | SecretSource    | SecureSettings is a list of references to Kubernetes secrets containing sensitive configuration options for APM Server.                               |
| serviceAccountName | string          | ServiceAccountName 用于检查在不同的命名空间从目前的资源访问资源(eg. Elasticsearch). 只能用于 ECK 是执行上的引用 RBAC.                                 |

[1]: https://www.elastic.co/guide/en/apm/server/current/configuring-howto-apm-server.html
