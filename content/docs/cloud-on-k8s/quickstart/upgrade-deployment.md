---
title: "升级部署"
linkTitle: ""
weight: 4
---

您可以添加和修改原始集群规范的大部分元素 前提是 它们可转化为底层 Kubernetes 资源的有效转化 (例如, 现有的卷权利要求不能被调整大小).
运营商将试图以最小的中断将更改应用于现有集群.
您应该确保 Kubernetes 集群有足够的资源来适应这些变化 (额外的存储空间, 足够的内存和 CPU 资源暂时旋转起来新的 pods 等.).

例如, 你可以增长群集到三个 Elasticsearch 节点:

```sh
cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
  - name: default
    count: 3
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
EOF
```
