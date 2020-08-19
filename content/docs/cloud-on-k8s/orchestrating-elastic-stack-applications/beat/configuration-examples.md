---
title: "配置举例"
linkTitle: ""
weight: 3
description: >
  下面你可以找到 manifests 该地址的一些常见的使用情况 并且可以成为你的起点 在使用ECK探索`Beats`部署 .
---

这些 manifests 是自包含的 和在任何非安全 Kubernetes 集群工作外的开箱 .
它们都含有三节点集群 Elasticsearch 和单 Kibana 实例.
所有`Beat`配置建立 Kibana 仪表板 如果他们是可用于给定`Beat`以及所有必需的 RBAC 资源.

这些实施例仅用于说明目的 而不应被认为是生产就绪.

有些例子使用 node.store.allow_mmap: false 配置值，以避免在配置存储器映射设置 在底层主机上.
这可能对你的 Elasticsearch 簇显著的性能影响 而不应在生产中使用没有仔细考虑.
想要查询更多的信息参见 https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-virtual-memory.html .

## Metricbeat 监测 Kubernetes

```sh
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/1.2/config/recipes/beats/metricbeat_hosts.yaml
```

部署 Metricbeat 作为 DaemonSet 监控主机资源使用 (CPU, memory, network, filesystem) 和 Kubernetes 资源 (Nodes, Pods, Containers, Volumes).

## Filebeat 与自动发现

```sh
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/1.2/config/recipes/beats/filebeat_autodiscover.yaml
```

部署 Filebeat 作为 DaemonSet 与自动发现功能启用.
它会在每一个命名空间从`pods`收集日志并将它们加载到所连 ​​ 接的 Elasticsearch 簇.

## Filebeat 与自动发现的元数据

```sh
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/1.2/config/recipes/beats/filebeat_autodiscover_by_metadata.yaml
```

部署 Filebeat 作为 DaemonSet 与自动发现功能启用.
从`pods`符合以下条件的日志 将被运到所连接的 Elasticsearch 簇:

- Pod 在`log-namespace`命名空间
- Pod 有 log-label: "true" 标签

## Filebeat 没有自动发现

```sh
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/1.2/config/recipes/beats/filebeat_no_autodiscover.yaml
```

部署 Filebeat 作为 DaemonSet 在禁用自动发现功能.
使用在主机上整个日志目录作为输入源.
此配置不需要任何 RBAC 资源 因为没有 Kubernetes APIs 使用.

## Elasticsearch 和 Kibana Stack 监控

```sh
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/1.2/config/recipes/beats/stack_monitoring.yaml
```

为 Elasticsearch 和 Kibana 堆 监控部署 Metricbeat 配置 和 Filebeat 使用自动发现.
部署 一个被监视 Elasticsearch 簇和一个监测 Elasticsearch 簇.
您可以在监控群集的 Kibana 访问堆栈监控应用 .
注意: 在这个例子中, TLS 验证是禁用 当 Metricbeat 与被监视簇连通, 这是不安全的，并且不应在生产中使用.
这可以通过使用自定义证书和配置 Metricbeat 验证他们解决.

## Heartbeat 监控 Elasticsearch 和 Kibana 健康

```sh
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/1.2/config/recipes/beats/heartbeat_es_kb_health.yaml
```

部署心跳为单`Pod`部署 监控 Elasticsearch 和 Kibana 的健康 通过 TCP 探测他们的服务端点.
需要注意的是`Heartbeat`预期 Eelasticsearch 和 Kibana 部署在默认的命名空间.

## Auditbeat

```sh
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/1.2/config/recipes/beats/auditbeat_hosts.yaml
```

部署 Auditbeat 作为 DaemonSet 该检查文件的完整性和审计文件操作 在主机系统上.

## Journalbeat

```sh
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/1.2/config/recipes/beats/journalbeat_hosts.yaml
```

部署 Journalbeat 作为 DaemonSet 从 systemd journals 传输数据.

## Packetbeat 监控 DNS 和 HTTP 传输

```sh
kubectl apply -f https://raw.githubusercontent.com/elastic/cloud-on-k8s/1.2/config/recipes/beats/packetbeat_dns_http.yaml
```

部署 Packetbeat 作为 DaemonSet 监控 DNS 在端口 53 和 HTTP(S) 流量 在端口 80, 8000, 8080 和 9200.
