---
title: "VM堆大小"
linkTitle: ""
weight: 1
description: >
  要改变Elasticsearch的堆大小, 在 podTemplate 里设置 ES_JAVA_OPTS 环境变量.
---

它还强烈建议设置在同一时间的资源请求和限制以确保 pod 得到 Kubernetes 集群内分配足够的资源.
见[设置 Elasticsearch 计算资源](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-managing-compute-resources.html#k8s-compute-resources-elasticsearch)示例获取更多信息。

如果 ES_JAVA_OPTS 没有定义, 1GI 的 Elasticsearch 默认堆大小将生效.

也可以看看: [有关设置堆大小 Elasticsearch 文档](https://www.elastic.co/guide/en/elasticsearch/reference/current/heap-size.html)
