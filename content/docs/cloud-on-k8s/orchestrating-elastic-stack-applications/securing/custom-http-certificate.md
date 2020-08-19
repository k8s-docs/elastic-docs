---
title: "自定义HTTP证书"
linkTitle: ""
weight: 1
description: >
  您可以提供自己的 CA 和证书，而不是自签名证书用一个 Kubernetes secret 通过 HTTPS 连接到 Elasticsearch .
---

该证书必须在 tls.crt 存储和私钥必须在 tls.key 存储.
如果您的证书不是由一个著名的 CA 颁发, 你必须包括在 ca.crt 信任链，以及.

你需要引用 secret 名 包含 TLS 私钥 和证书 (和任选的信任链), 在 spec.http.tls.certificate 里的部分.

```yaml
spec:
  http:
    tls:
      certificate:
        secretName: quickstart-es-cert
```

## 使用 OpenSSL 证书自定义自签名

This example illustrates how to create your own self-signed certificate for the quickstart Elasticsearch cluster using the OpenSSL command line utility. Note the subject alternative name (SAN) entry for quickstart-es-http.default.svc.

```sh
$ openssl req -x509 -sha256 -nodes -newkey rsa:4096 -days 365 -subj "/CN=quickstart-es-http" -addext "subjectAltName=DNS:quickstart-es-http.default.svc" -keyout tls.key -out tls.crt
$ kubectl create secret generic quickstart-es-cert --from-file=ca.crt=tls.crt --from-file=tls.crt=tls.crt --from-file=tls.key=tls.key
```

## 使用 cert-manager 自定义自签名证书

这个例子说明如何用一个 cert-manager 自签署发行为快速启动 Elasticsearch 集群发出自签名证书.

```yaml
---
apiVersion: cert-manager.io/v1alpha2
kind: Issuer
metadata:
  name: selfsigned-issuer
spec:
  selfSigned: {}
---
apiVersion: cert-manager.io/v1alpha2
kind: Certificate
metadata:
  name: quickstart-es-cert
spec:
  isCA: true
  dnsNames:
    - quickstart-es-http
    - quickstart-es-http.default.svc
    - quickstart-es-http.default.svc.cluster.local
  issuerRef:
    kind: Issuer
    name: selfsigned-issuer
  secretName: quickstart-es-cert
```
