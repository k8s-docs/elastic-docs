---
title: "禁用webhook"
linkTitle: ""
weight: 4
description: >
  要禁用webhook, 设置 [enable-webhook](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-operator-config.html) 运营商配置标志 to false 并取出 ValidatingWebhookConfiguration 命名 elastic-webhook.k8s.elastic.co:
---

```sh
kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io elastic-webhook.k8s.elastic.co
```
