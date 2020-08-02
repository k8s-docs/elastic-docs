---
title: "节点配置"
linkTitle: ""
weight: 2
---

在`elasticsearch.yml`配置文件中的任何设置定义也可以定义为 在`spec.nodeSets[?].config`部分的 Elasticsearch 的集合节点设置.

```yaml
spec:
  nodeSets:
    - name: masters
      count: 3
      config:
        node.master: true
        node.data: false
        node.ingest: false
        node.ml: false
        xpack.ml.enabled: true
        node.remote_cluster_client: false
    - name: data
      count: 10
      config:
        node.master: false
        node.data: true
        node.ingest: true
        node.ml: true
        node.remote_cluster_client: false
```

有关 Elasticsearch 设置的详细信息，请参阅[配置 Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/current/settings.html).
