---
title: "常见问题"
linkTitle: ""
weight: 1
---

## 使用 OOMKilled 启动操作崩溃

在非常大的拥有数百资源 Kubernetes 集群 (pods, secrets, config maps, 等等), 运营商可能无法启动 其`pod`被杀 用 OOMKilled 消息.
这是在顶部的控制器的运行时框架，它的运营商建立的一个问题.
即使操作者只关心在自身资源, 框架代码需要收集有关 Kubernetes 集群中的所有相关资源的信息 为了提供群集状态的过滤视图 由运营商要求.
在非常大的集群, 此信息收集最多可以使用大量的内存 并超过了运营商`pod`定义的默认资源限制.

操作员`pod`默认内存限制设置为 512 MIB.
您可以增加（或减少）这个限制以适合您的群集的值如下:

```sh
kubectl patch sts elastic-operator -n elastic-system -p '{"spec":{"template":{"spec":{"containers":[{"name":"manager", "resources":{"limits":{"memory":"768Mi"}}}]}}}}'
```

## 提交 manifest 时超时

当提交 ECK 资源`manifest`, 您可能会遇到类似如下的错误信息:

```
Error from server (Timeout): error when creating "elasticsearch.yaml": Timeout: request did not complete within requested timeout 30s
```

此错误通常是一个问题的指示 与所述验证`webhook`连通.
如果你是一个私人谷歌 Kubernetes 引擎（GKE）集群上运行 ECK, 您可能需要添加防火墙规则允许来自 API 服务器的端口 9443.
另一个可能的原因是失败，如果一个严格的网络策略生效.
请参阅`webhook`故障排除文档 有关详细信息和解决方法.

## 使用业主参考复制的秘密

通过复制 ECK 产生的 Elasticsearch 秘密(例如，证书颁发机构或弹性用户) 到另一个命名空间 批发可以触发 Kubernetes 错误 它可以删除所有 Elasticsearch 相关资源(例如，数据量).
为了避免这种情况,复制命名空间之间的秘密时, 去除 metadata.ownerReferences 部分.
例如，源秘密可能是:

```sh
kubectl get secret quickstart-es-elastic-user -o yaml
apiVersion: v1
data:
  elastic: NGw2Q2REMjgwajZrMVRRS0hxUDVUUTU0
kind: Secret
metadata:
  creationTimestamp: "2020-06-09T19:11:41Z"
  labels:
    common.k8s.elastic.co/type: elasticsearch
    eck.k8s.elastic.co/credentials: "true"
    elasticsearch.k8s.elastic.co/cluster-name: quickstart
  name: quickstart-es-elastic-user
  namespace: default
  ownerReferences:
  - apiVersion: elasticsearch.k8s.elastic.co/v1
    blockOwnerDeletion: true
    controller: true
    kind: Elasticsearch
    name: quickstart
    uid: c7a9b436-aa07-4341-a2cc-b33b3dfcbe29
  resourceVersion: "13048277"
  selfLink: /api/v1/namespaces/default/secrets/quickstart-es-elastic-user
  uid: 04cdf334-77d3-4de6-a2e8-7a2b23366a27
type: Opaque
```

将其复制到不同的命名空间, 剥离 metadata.ownerReferences 字段 还有 object-specific 数据:

```yaml
apiVersion: v1
data:
  elastic: NGw2Q2REMjgwajZrMVRRS0hxUDVUUTU0
kind: Secret
metadata:
  labels:
    common.k8s.elastic.co/type: elasticsearch
    eck.k8s.elastic.co/credentials: "true"
    elasticsearch.k8s.elastic.co/cluster-name: quickstart
  name: quickstart-es-elastic-user
  namespace: default
type: Opaque
```

如果不这样做会导致数据丢失.
