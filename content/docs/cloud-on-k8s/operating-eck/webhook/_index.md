---
title: "配置验证webhook"
linkTitle: "webhook"
weight: 2
description: >
  一个进行验证 webhook 提供 Elasticsearch 资源附加验证: 它提供了您所提交的 Elasticsearch 清单即时反馈,
  让您捕捉错误的时候了 之前 ECK 甚至试图以满足您的要求.
---

## Architecture

webhook 是由 4 个主要组件.这里是他们每个人的简要说明，了解它们是如何相互作用, 他们的命名，以及它们是如何管理.

1. ValidatingWebhookConfiguration 对象，定义验证 webhook, 瞄准正确的 webhook 路径和资源. It must be created before starting the operator. The caBundle field can be automatically managed as part of the automatic certificate management (see below).
2. A Kubernetes Service is used to expose the validating server, named elastic-webhook-server. It is in the same Namespace as the webhook server.
3. A webhook server that actually validates the submitted resources. In ECK it is the operator itself when it is configured with the webhook enabled. See Configuring ECK for more information about the enable-webhook flag.
4. A Secret containing the required certificates to secure the connection between the API server and the webhook server. Like the ValidatingWebhookConfiguration, it must be created before starting the operator, even if it is empty. By default its name is elastic-webhook-server-cert. The content of this Secret and the lifecycle of the certificates are automatically managed for you. ECK generates a dedicated and separate certificate authority and ensures that all components are rotated before the expiration date. The certificate authority is also used to configure the caBundle field of the ValidatingWebhookConfiguration. You can disable this feature if you want to manage the certificates yourself or with cert-manager. See this example.
