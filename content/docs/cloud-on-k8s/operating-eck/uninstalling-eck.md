---
title: "卸载 ECK"
linkTitle: ""
weight: 7
---

在卸载操作之前, 删除所有弹性资源的所有命名空间:

```sh
kubectl get namespaces --no-headers -o custom-columns=:metadata.name \
 | xargs -n1 kubectl delete elastic --all -n
```

这将删除所有底层 Elasticsearch, Kibana, 和 APM 服务器资源 (pods, secrets, services, etc.).

然后，你可以卸载操作:

```sh
kubectl delete -f https://download.elastic.co/downloads/eck/1.2.0/all-in-one.yaml
```
