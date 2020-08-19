---
title: "词汇表"
linkTitle: ""
weight: 2
description: >
  本词汇表补充Kubernetes词汇和覆盖在Kubernetes（ECK）文档上在弹性云使用的醒目.
---

### CA - Certificate Authority

实体颁发数字证书来验证身份在网络上。

### Cluster

根据上下文可以指一个 Elasticsearch 集群或 Kubernetes 集群。

### CRD - Custom Resource Definition

ECK 扩展了顶层要求的 Kubernetes API 来允许用户部署和管理 Elasticsearch，Kibana 和 APM 服务器资源就像他们会用做内置 Kubernetes 资源.

### ECK - Elastic Cloud on Kubernetes

Kubernetes 运营商编排 Kubernetes 上 Elasticsearch, Kibana 和 APM 服务器部署 .

### EKS - Elastic Kubernetes Service

通过亚马逊网络服务（AWS）提供托管服务 Kubernetes.

### GCS - Google Cloud Storage

由谷歌云平台提供块存储服务 (GCP).

### GKE - Google Kubernetes Engine

由谷歌云平台提供的托管服务 Kubernetes (GCP).

### K8s

“Kubernetes” 的缩写形式（numeronym） 从用“8”代替“ubernete”衍生.

### Node

根据上下文可以指一个 Elasticsearch 节点或 Kubernetes 节点.
一个 ECK Elasticsearch 节点映射到 Kubernetes`Pod` 由此可得安排到任何可用的 Kubernetes 节点 能够满足在`pod`模板中定义的资源需求和`node`限制.

### NodeSet

一组 Elasticsearch`nodes`共享同一 Elasticsearch 配置和`Kubernetes Pod`模板.
多`NodeSets`可以在 Elasticsearch CRD 定义实现了集群拓扑由 of Elasticsearch`nodes`组 不同节点的角色，资源需求和硬件配置 (Kubernetes node 限制).

### OpenShift

一个红帽的 Kubernetes 平台 .

### Operator

Kubernetes 的设计模式管理自定义资源.
在 Kubernetes 上 ECK 实现操作模式来管理 Elasticsearch，Kibana 和 APM 服务器资源 .

### PDB - Pod Disruption Budget

Pod 破坏预算.

### PVC - Pod Disruption Budget

持久卷权利要求.

### QoS - Quality of Service

当一个 Kubernetes 簇是在重负载下, 该 Kubernetes 调度让`pod`驱逐决定 基于个人`pods`的 QoS 等级 .
管理计算资源解释了如何定义 QoS 等级为 Elasticsearch, Kibana 和 APM 服务器 pods.

### RBAC - Role-based Access Control

Kubernetes 上的安全机制 其中访问群集资源 仅限于校长 有合适的角色.
想要查询更多的信息参考 https://kubernetes.io/docs/reference/access-authn-authz/rbac/ .
