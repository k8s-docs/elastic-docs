---
title: "HTTP配置"
linkTitle: ""
weight: 1
---

## 负载均衡器设置和 TLS SANs

By default a ClusterIP service is created and associated with the Kibana deployment. You may want to expose Kibana externally with a load balancer. In which case you may also want to include a custom DNS name or IP in the self-generated certificate.

```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
name: kibana-sample
spec:
version: 7.8.1
count: 1
elasticsearchRef:
name: "elasticsearch-sample"
http:
service:
spec:
type: LoadBalancer # default is ClusterIP
tls:
selfSignedCertificate:
subjectAltNames: - ip: 1.2.3.4 - dns: kibana.example.com
```

## 提供您自己的证书

If you want to use your own certificate, the required configuration is identical to Elasticsearch. See: Custom HTTP certificate.

## 禁用 TLS

You can disable the generation of the self-signed certificate and hence disable TLS.

```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
name: kibana-sample
spec:
version: 7.8.1
count: 1
elasticsearchRef:
name: "elasticsearch-sample"
http:
tls:
selfSignedCertificate:
disabled: true
```
