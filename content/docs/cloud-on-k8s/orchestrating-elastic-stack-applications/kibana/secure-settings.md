---
title: "安全设置"
linkTitle: ""
weight: 1
---

Similar to Elasticsearch, you can use Kubernetes secrets to manage secure settings for Kibana as well.

For example, you can define a custom encryption key for Kibana as follows:

Create a secret containing the desired setting:

```sh
kubectl create secret generic kibana-secret-settings \
 --from-literal=xpack.security.encryptionKey=94d2263b1ead716ae228277049f19975aff864fb4fcfe419c95123c1e90938cd
```

Add a reference to the secret in the secureSettings section:

```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
name: kibana-sample
spec:
version: 7.8.1
count: 3
elasticsearchRef:
name: "elasticsearch-sample"
secureSettings:

- secretName: kibana-secret-settings
```
