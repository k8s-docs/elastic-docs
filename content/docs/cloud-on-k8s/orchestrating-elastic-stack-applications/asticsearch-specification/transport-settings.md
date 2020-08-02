---
title: "传输设置"
linkTitle: ""
weight: 5
---

在 Elasticsearch 传输模块用于 在集群内的节点之间的内部通信以及远程簇之间通信.
详情请参阅 Elasticsearch 文档。

在 spec.transport.service. 部分, 您可以更改用于公开 Elasticsearch 传输模块的 Kubernetes 服务:

```yaml
spec:
  transport:
    service:
      metadata:
        labels:
          my-custom: label
      spec:
        type: LoadBalancer
```

检查 Kubernetes 出版服务 (ServiceTypes) 目前可用.

请注意，当您更改服务的 clusterIP 设置, ECK 将删除并重新创建服务作为 clusterIP 是一个不可变字段.
这通常不会对连接性的影响 作为传输模块采用长寿命的 TCP 连接, 但可能会导致 Elasticsearch 节点之间的小网络中断.
