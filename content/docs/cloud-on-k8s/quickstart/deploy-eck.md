---
title: "在您的Kubernetes集群部署ECK"
linkTitle: "部署ECK"
weight: 1
---

> 首先阅读[升级说明](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-upgrading-eck.html)，如果你正试图升级现有 ECK 部署。

> 如果您使用的 GKE，请确保您的用户群管理员的权限。 有关详细信息，请参阅[前提条件对 GKE 使用 Kubernetes RBAC](https://cloud.google.com/kubernetes-engine/docs/how-to/role-based-access-control#iam-rolebinding-bootstrap)。

> 如果你正在使用亚马逊 EKS, 确保 Kubernetes 控制平面被允许在端口 443 上与 Kubernetes 节点进行通信.
> 这需要与验证网络挂接通信。
> 欲了解更多信息，请参见[推荐入站流量](https://docs.aws.amazon.com/eks/latest/userguide/sec-group-reqs.html)。

1. 安装[自定义资源](https://kubernetes.io/docs/concepts/extend-kubernetes/api-extension/custom-resources/)的定义和运营商，其 RBAC 规则:

   ```sh
   kubectl apply -f https://download.elastic.co/downloads/eck/1.2.0/all-in-one.yaml
   ```

2. 监控操作日志:

   ```sh
   kubectl -n elastic-system logs -f statefulset.apps/elastic-operator
   ```
