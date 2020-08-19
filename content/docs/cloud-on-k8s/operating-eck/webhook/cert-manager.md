---
title: "使用cert-manager管理webhook证书 "
linkTitle: "certmanager"
weight: 1
description: >
  如果ECK当前正在运行, 首先确保自动证书管理功能被禁用.
  去做这个, 添加 --manage-webhook-certs=false 标记给操作员部署清单.
---

然后, 如上所述在证书管理器文档中安装[证书管理器](https://docs.cert-manager.io/en/latest/getting-started/install/) v0.11+ .

此示例演示如何创建的所有资源 一个 webhook 要求的功能。
webhook.yaml

```yaml
# this configures
# - a self signed cert-manager issuer
# - a service to point to the webhook
# - a self signed certificate for the webhook service
# - a validating webhook configuration
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: selfsigned-issuer
  namespace: elastic-system
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: elastic-webhook
  namespace: elastic-system
spec:
  commonName: elastic-webhook.elastic-system.svc
  dnsNames:
    - elastic-webhook.elastic-system.svc.cluster.local
    - elastic-webhook.elastic-system.svc
  issuerRef:
    kind: Issuer
    name: selfsigned-issuer
  secretName: elastic-webhook-server-cert
---
apiVersion: v1
kind: Service
metadata:
  name: elastic-webhook-server
  namespace: elastic-system
spec:
  ports:
    - port: 443
      protocol: TCP
      targetPort: 9443
      name: https-webhook
  selector:
    control-plane: elastic-operator
  sessionAffinity: None
  type: ClusterIP
---
apiVersion: admissionregistration.k8s.io/v1beta1
kind: ValidatingWebhookConfiguration
metadata:
  name: elastic-webhook.k8s.elastic.co
  annotations:
    cert-manager.io/inject-ca-from: elastic-system/elastic-webhook
webhooks:
  - clientConfig:
      caBundle: Cg==
      service:
        name: elastic-webhook
        namespace: elastic-system
        # this is the path controller-runtime automatically generates
        path: /validate-elasticsearch-k8s-elastic-co-v1-elasticsearch
    failurePolicy: Ignore
    name: elastic-es-validation-v1.k8s.elastic.co
    sideEffects: None
    rules:
      - apiGroups:
          - elasticsearch.k8s.elastic.co
        apiVersions:
          - v1
        operations:
          - CREATE
          - UPDATE
        resources:
          - elasticsearches
```

```
kubectl apply -f webhook.yaml
```

> 这个例子假定在 elastic-system 命名空间里面您已经安装了运营商 .
