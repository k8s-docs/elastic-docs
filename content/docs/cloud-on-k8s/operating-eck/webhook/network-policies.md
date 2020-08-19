---
title: "网络策略"
linkTitle: ""
weight: 3
description: >
  Webhooks 需要 Kubernetes API 服务器和运营商之间的网络连接.
---

如果一个 Elasticsearch 资源创建超时并产生错误消息类似于以下, 那么 Kubernetes API 服务器可能无法连接到 webhook 验证清单.

```
Error from server (Timeout): error when creating "elasticsearch.yaml": Timeout: request did not complete within requested timeout 30s
```

如果您收到此错误, 尝试重新运行具有较高的请求超时的命令如下:

```sh
kubectl --request-timeout=1m apply -f elasticsearch.yaml
```

作为默认`webhook`的`failurePolicy`被忽略, 上面的命令应该约 30 秒后成功.
这是一个迹象该 API 服务器无法联系`webhook`服务器并具有已成验证 当创建资源.
有可能的 一个`network`策略阻止所有传入的请求到`webhook`服务器.
咨询系统管理员，以确定是否是这样的话, 并创建相应的策略，允许通信 在 Kubernetes API 服务器和`webhook`服务器之间.
例如, 下面的网络策略只是打开了`webhook`端口到世界各地:

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-webhook-access-from-any
  namespace: elastic-system
spec:
  podSelector:
    matchLabels:
      control-plane: elastic-operator
  ingress:
    - from: []
      ports:
        - port: 9443
```

如果你想限制`webhook`只能访问 Kubernetes API 服务器, 你必须知道的 API 服务器的 IP 地址, 你可以通过这个命令来获得:

```sh
kubectl cluster-info | grep master
```

假设 API 服务器的 IP 地址是 10.1.0.1, 以下策略限制`webhook`访问刚刚 API 服务器.

```yaml
kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
  name: allow-webhook-access-from-apiserver
  namespace: elastic-system
spec:
  podSelector:
    matchLabels:
      control-plane: elastic-operator
  ingress:
    - from:
        - ipBlock:
            cidr: 10.1.0.1/32
      ports:
        - port: 9443
```
