---
title: "禁用webhook"
linkTitle: ""
weight: 4
description: >
  To disable the webhook, set the enable-webhook operator configuration flag to false and remove the ValidatingWebhookConfiguration named elastic-webhook.k8s.elastic.co:
---

kubectl delete validatingwebhookconfigurations.admissionregistration.k8s.io elastic-webhook.k8s.elastic.co
