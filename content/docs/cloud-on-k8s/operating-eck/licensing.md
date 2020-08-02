---
title: "在ECK里管理许可证"
linkTitle: "管理许可证"
weight: 3
description: >
  ECK 只在两种许可层级提供: 基本和企业. 类似于 Elastic 堆栈, 用户可以下载和使用 ECK 有一个免费的基本许可证. 基本许可证的用户可以从 GitHub 或者通过我们的社区获得支撑. 已付费企业订购 需要接合 Elastic 支持团队.
---

当您安装 ECK 的默认分配, 您会收到一个基本的许可证.您通过 ECK 管理任何 Elastic 栈的应用也将基本许可. 去https://www.elastic.co/subscriptions查看其功能都包含在免费的基本许可证.

更多细节, 见 Elastic 订阅.

在本节中，您将学习如何:

- Start a trial
- Add a license
- Update your license
- Get usage data

## 开始试用

If you want to try the features included in the Enterprise subscription, you can start a 30-day trial. To start a trial create a Kubernetes secret as shown below. Note that it must be in the same namespace as the operator:

```sh
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: Secret
metadata:
  name: eck-trial-license
  namespace: elastic-system
  labels:
    license.k8s.elastic.co/type: enterprise_trial
  annotations:
    elastic.co/eula: accepted
EOF
```

By setting this annotation to accepted you are expressing that you have accepted the Elastic EULA which can be found at https://www.elastic.co/eula.

You can initiate a trial only if a trial has not been previously activated.

At the end of the trial period, the Platinum and Enterprise features operate in a degraded mode. You can revert to a Basic license, extend the trial, or purchase an Enterprise subscription.

## 添加许可证

If you have a valid Enterprise subscription or a trial license extension, you will receive a license as a JSON file.

ECK will only accept Enterprise licenses. You can not apply a single Platinum or Gold cluster license to ECK.

To add the license to your ECK installation, create a Kubernetes secret of the following form:

```yaml
apiVersion: v1
kind: Secret
metadata:
  labels:
    license.k8s.elastic.co/scope: operator
  name: eck-license
type: Opaque
data:
  license: "JSON license in base64 format"
```

This label is required for ECK to identify your license secret.

The license file can have any name.

You can easily create this secret using kubectl built-in support for secrets. Note that it must be in the same namespace as the operator:

```sh
kubectl create secret generic eck-license --from-file=my-license-file.json -n elastic-system
kubectl label secret eck-license "license.k8s.elastic.co/scope"=operator -n elastic-system
```

After you install a license into ECK, all the Elastic stack applications you manage with ECK will have all Platinum and Enterprise features enabled. Applications created before you installed the license are upgraded to Platinum or Enterprise features without interruption of service after a short delay.

## 更新许可证

Before your current Enterprise license expires, you will receive a new Enterprise license from Elastic (provided that your subscription is valid).

To avoid any unintended downgrade of individual Elasticsearch clusters to a Basic license while installing the new license, we recommend to install the new Enterprise license as a new Kubernetes secret next to your existing Enterprise license. Just replace eck-license with a different name from the examples above. ECK will use the correct license automatically.

Once you have created the new license secret you can safely delete the old license secret.

## 获取使用数据

The operator periodically writes the total amount of Elastic resources under management to a configmap named elastic-licensing, which is in the same namespace as the operator. Here is an example of retrieving the data:

```sh
kubectl -n elastic-system get configmap elastic-licensing -o json | jq .data
{
  "eck_license_level": "enterprise",
  "enterprise_resource_units": "1",
  "max_enterprise_resource_units": "10",
  "timestamp": "2020-01-03T23:38:20Z",
  "total_managed_memory": "3.22GB"
}
```

If the operator metrics endpoint is enabled via the --metrics-port flag (see Configure ECK), license usage data will be included in the reported metrics.

```sh
curl "$ECK_METRICS_ENDPOINT" | grep elastic_licensing
# HELP elastic_licensing_enterprise_resource_units_total Total enterprise resource units used
# TYPE elastic_licensing_enterprise_resource_units_total gauge
elastic_licensing_enterprise_resource_units_total{license_level="basic"} 6
# HELP elastic_licensing_memory_gigabytes_total Total memory used in GB
# TYPE elastic_licensing_memory_gigabytes_total gauge
elastic_licensing_memory_gigabytes_total{license_level="basic"} 357.01915648
```
