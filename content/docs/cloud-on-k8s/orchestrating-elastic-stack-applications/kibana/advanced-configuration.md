---
title: "高级配置"
linkTitle: ""
weight: 1
---

If you already looked at the Elasticsearch on ECK documentation, then concepts and ideas described here might sound familiar to you. This is because the resource definitions in ECK share the same philosophy when it comes to:

Customizing the Pod configuration
Customizing the product configuration
Managing HTTP settings
Using secure settings

## Pod 配置

You can customize the Kibana pod using a Pod template.

The following example demonstrates how to create a Kibana deployment with custom node affinity and resource limits.

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
podTemplate:
spec:
containers: - name: kibana
resources:
requests:
memory: 1Gi
cpu: 0.5
limits:
memory: 2Gi
cpu: 2
nodeSelector:
type: frontend
```

The name of the container in the pod template must be kibana.

See Set compute resources for Kibana and APM Server for more information.

## Kibana 配置

You can add your own Kibana settings to the spec.config section.

The following example demonstrates how to set the elasticsearch.requestHeadersWhitelist configuration option:

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
config:
elasticsearch.requestHeadersWhitelist: - authorization
```

## 向外扩展 Kibana 部署

You may want to deploy more than one instance of Kibana. In this case all the instances must share the same encryption key. If you do not set one, the operator will generate one for you. If you would like to set your own encryption key, this can be done by setting the xpack.security.encryptionKey property using a secure setting as described in the next section.

Note that while most reconfigurations of your Kibana instances will be carried out in rolling upgrade fashion, all version upgrades will cause Kibana downtime. This is due to the requirement to run only a single version of Kibana at any given time.
