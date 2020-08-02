---
title: "卷要求的模板"
linkTitle: "卷模板"
weight: 3
description: >
  默认, 运营商创建一个带在 Elasticsearch 集群每个 pod 1GI 的容量的 PersistentVolumeClaim 为防止意外 pod 缺失的情况下，数据丢失.
---

对于生产工作负载, 你应该定义自己的所需的存储容量卷需求模板和(可选)Kubernetes 存储类与持久卷关联.
卷要求的名称必须始终为 `elasticsearch-data`.

```yaml
spec:
  nodeSets:
    - name: default
      count: 3
      volumeClaimTemplates:
        - metadata:
            name: elasticsearch-data
          spec:
            accessModes:
              - ReadWriteOnce
            resources:
              requests:
                storage: 5Gi
            storageClassName: standard
```

ECK 自动删除 PersistentVolumeClaim 资源 如果他们不被任何 Elasticsearch 节点所需要.
相应 PersistentVolume 可保留, 这取决于配置的存储类回收政策.

根据 Kubernetes 配置和底层文件系统, 他们在创建后一些持久卷不能调整大小.
当你定义卷声明, 考虑未来的存储需求 和 请确保您有足够的空间来支持预期增长.

如果你不关心数据丢失, 您可以为 Elasticsearch 数据使用 emptyDir 卷 :

```yaml
spec:
  nodeSets:
    - name: data
      count: 10
      podTemplate:
        spec:
          volumes:
            - name: elasticsearch-data
              emptyDir: {}
```

使用 emptyDir 不推荐 由于数据永久丢失的可能性很高.
