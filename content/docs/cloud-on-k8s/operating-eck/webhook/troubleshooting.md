---
title: "故障排除"
linkTitle: ""
weight: 2
description: >
  在启动时, 在运营商部署承认`webhook` 它指向运营商的服务.
---

如果这是不可访问, 您可能会看到您的 Kubernetes API 服务器日志错误 这表明它不能达到服务.
一个常见的原因可能是操作者`pods`都未能启动由于某种原因, 或者，所述控制平面被从运算符'pod`通过一些机制分离 (对于通过网络策略实例或外部运行控制平面的问题 #869 及问题 #1369).

有关疑难解答, 你可以改变`webhook`的 failurePolicy 配置 为 Fail, 这将导致创作和更新出错了 如果有错误接触`webhook`.

在 GKE 私人集群, 创建或更新弹性资源的请求可能需要很长的时间来完成或超时.
在这种情况下, 你必须添加防火墙规则允许端口 9443 从 API 服务器，以便它可以联系`webhook`.
详情请参见上添加规则的 GKE 文档和 Kubernetes 问题.

请参阅网络策略有关可能阻止 Kubernetes API 服务器和`webhook`服务器之间的通信网络策略的详细信息.

## 验证失败

如果验证`webhook`阻止您更改由于未知领域验证就像在下面的例子, 您可以通过从您的资源移除`kubectl.kubernetes.io/last-applied-configuration` 注解强制`webhook`忽略它.

```
admission webhook "elastic-es-validation-v1.k8s.elastic.co" denied the request: Elasticsearch.elasticsearch.k8s.elastic.co "quickstart" is invalid: some-misspelled-field: Invalid value: "some-misspelled-field": some-misspelled-field field found in the kubectl.kubernetes.io/last-applied-configuration annotation is unknown
```
