---
title: "检出样本"
linkTitle: ""
weight: 5
---

您可以在[项目档案库](https://github.com/elastic/cloud-on-k8s/tree/1.2/config/samples)找出一组样本资源。

对于每个 CustomResourceDefinition 的完整描述 (CRD), 参考 [API 参考](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-api-reference.html) 要么 查看[项目库](https://github.com/elastic/cloud-on-k8s/tree/1.2/config/crds)的 CRD 文件.
您还可以检索有关来自集群中 CRD 信息. 例如, 使用如下描述 Elasticsearch CRD 规范:

```sh
kubectl describe crd elasticsearch
```
