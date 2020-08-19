---
title: "连接到一个Elasticsearch集群"
linkTitle: ""
weight: 1
---

## 连接到由 ECK 管理的 Elasticsearch 集群

It is quite straightforward to connect a Kibana instance to an Elasticsearch cluster managed by ECK:

```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
name: quickstart
spec:
version: 7.8.1
count: 1
elasticsearchRef:
name: quickstart
namespace: default
```

namespace is optional if the Elasticsearch cluster is running in the same namespace as Kibana.

Any Kibana can reference (and thus access) any Elasticsearch instance as long as they both are in namespaces that are watched by the same ECK instance. ECK will copy the required Secret from Elasticsearch to Kibana namespace. If this behavior is not desired, more than one ECK instance can be deployed. Kibana won’t be able to automatically connect to Elasticsearch (through elasticsearchRef) in a namespace managed by a different ECK instance.

The Kibana configuration file is automatically setup by ECK to establish a secure connection to Elasticsearch.

## 连接到不是 ECK 管理的 Elasticsearch 集群

It is also possible to configure Kibana to connect to an Elasticsearch cluster that is being managed by a different installation of ECK or running outside the Kubernetes cluster. In this case, you need to know the IP address or URL of the Elasticsearch cluster and a valid username and password pair to access the cluster.

Use the secure settings mechanism to securely store the credentials of the external Elasticsearch cluster:

```sh
kubectl create secret generic kibana-elasticsearch-credentials --from-literal=elasticsearch.password=\$PASSWORD
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
name: kibana-sample
spec:
version: 7.8.1
count: 1
config:
elasticsearch.hosts: - https://elasticsearch.example.com:9200
elasticsearch.username: elastic
secureSettings: - secretName: kibana-elasticsearch-credentials
```

If the external Elasticsearch cluster is using a self-signed certificate, create a Kubernetes secret containing the CA certificate and mount it to the Kibana container as follows:

```yaml
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
name: kibana-sample
spec:
version: 7.8.1
count: 1
config:
elasticsearch.hosts: - https://elasticsearch-sample-es-http:9200
elasticsearch.username: elastic
elasticsearch.ssl.certificateAuthorities: /etc/certs/ca.crt
secureSettings: - secretName: kibana-elasticsearch-credentials
podTemplate:
spec:
volumes: - name: elasticsearch-certs
secret:
secretName: elasticsearch-certs-secret
containers: - name: kibana
volumeMounts: - name: elasticsearch-certs
mountPath: /etc/certs
readOnly: true
```
