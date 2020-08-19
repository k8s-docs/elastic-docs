---
title: "配置"
linkTitle: ""
weight: 3
---

## 升级企业搜索规范

Any setting can be changed in the Enterprise Search yaml specification, including version upgrades. ECK detects those changes and ensures a smooth rolling upgrade of all Enterprise Search Pods.

## 自定义企业级搜索的配置

ECK sets up a default Enterprise Search configuration. Customize it through the config element in the specification.

At a minimum, you must set ent_search.external_url to the desired URL.

```yaml
apiVersion: enterprisesearch.k8s.elastic.co/v1beta1
kind: EnterpriseSearch
metadata:
name: enterprise-search-quickstart
spec:
version: 7.8.1
count: 1
elasticsearchRef:
name: quickstart
config: # define the exposed URL at which users will reach Enterprise Search
ent_search.external_url: https://my-custom-domain:3002 # configure app search document size limit
app_search.engine.document_size.limit: 100kb
```

## 参考 Kubernetes 秘密 对于敏感的设置

Some sensitive settings are best stored in Kubernetes Secrets, referenced in the Enterprise Search specification.

This example sets up a Secret with SMTP credentials:

```yaml
apiVersion: enterprisesearch.k8s.elastic.co/v1beta1
kind: EnterpriseSearch
metadata:
name: enterprise-search-quickstart
spec:
version: 7.8.1
count: 1
elasticsearchRef:
name: quickstart
config:
ent_search.external_url: https://my-custom-domain:3002
configRef:
secretName: smtp-credentials

---

kind: Secret
apiVersion: v1
metadata:
name: smtp-credentials
stringData:
enterprise-search.yml: |-
email.account.enabled: true
email.account.smtp.auth: plain
email.account.smtp.starttls.enable: false
email.account.smtp.host: 127.0.0.1
email.account.smtp.port: 25
email.account.smtp.user: myuser
email.account.smtp.password: mypassword
email.account.email_defaults.from: my@email.com
```

ECK merges the content of config and configRef into a single internal Secret. In case of duplicate settings, the configRef secret has precedence.

## 自定义`Pod`模板

You can override the Enterprise Search Pods specification through the podTemplate element.

This example overrides the default 4Gi deployment to use 8Gi instead, and makes the deployment highly-available with 3 Pods:

```yaml
apiVersion: enterprisesearch.k8s.elastic.co/v1beta1
kind: EnterpriseSearch
metadata:
name: enterprise-search-quickstart
spec:
version: 7.8.1
count: 3
elasticsearchRef:
name: quickstart
podTemplate:
spec:
containers: - name: enterprise-search
resources:
requests:
cpu: 3
memory: 8Gi
limits:
memory: 8Gi
env: - name: JAVA_OPTS
value: -Xms7500m -Xmx7500m
```

## 揭露企业搜索

By default ECK manages self-signed TLS certificates to secure the connection to Enterprise Search. It also restricts the Kubernetes service to ClusterIP type that cannot be accessed publicly.

See how to access Elastic Stack services to customize TLS settings and expose the service.

When exposed outside the scope of localhost, make sure to set ent_search.external_url accordingly in the Enterprise Search configuration.

## 连接到外部 Elasticsearch 簇

The elasticsearchRef element allows ECK to automatically configure Enterprise Search to establish a secured connection to a managed Elasticsearch cluster.

Also, you can manually configure Enterprise Search to access any available Elasticsearch cluster:

```yaml
apiVersion: enterprisesearch.k8s.elastic.co/v1beta1
kind: EnterpriseSearch
metadata:
name: enterprise-search-quickstart
spec:
version: 7.8.1
count: 1
configRef:
secretName: elasticsearch-credentials

---
kind: Secret
apiVersion: v1
metadata:
name: elasticsearch-credentials
stringData:
enterprise-search.yml: |-
elasticsearch.host: <a href="https://elasticsearch-url:9200" class="ulink" target="_top">https://elasticsearch-url:9200</a>
elasticsearch.username: elastic
elasticsearch.password: my-password
elasticsearch.ssl.enabled: true
```
