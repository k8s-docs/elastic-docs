---
title: "使用cert-manager管理webhook证书 "
linkTitle: "certmanager"
weight: 1
description: >
  If ECK is currently running, first make sure that the automatic certificate management feature is disabled.
  To do this, add the --manage-webhook-certs=false flag to the operator deployment manifest.
---

Then, install cert-manager v0.11+ as described in the cert-manager documentation.

This example shows how to create all the resources that a webhook requires to function.

```sh
cat <<EOF | kubectl apply -f -
---
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
EOF
```

> This example assumes that you have installed the operator in the elastic-system namespace.
