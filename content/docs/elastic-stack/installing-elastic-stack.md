---
title: "安装弹性堆栈"
linkTitle: ""
weight: 2
---

When installing the Elastic Stack, you must use the same version across the entire stack. For example, if you are using Elasticsearch 7.8.1, you install Beats 7.8.1, APM Server 7.8.1, Elasticsearch Hadoop 7.8.1, Kibana 7.8.1, and Logstash 7.8.1.

If you’re upgrading an existing installation, see Upgrading the Elastic Stack for information about how to ensure compatibility with 7.8.1.

## 安装顺序

Install the Elastic Stack products you want to use in the following order:

1. Elasticsearch (install instructions)
2. Kibana (install)
3. Logstash (install)
4. Beats (install instructions)
5. APM Server (install instructions)
6. Elasticsearch Hadoop (install instructions)

Installing in this order ensures that the components each product depends on are in place.

## 安装在弹性云

在弹性云上 Elasticsearch 服务是来自 Elastic 提供官方托管 Elasticsearch 和 Kibana.
该 Elasticsearch 服务均可在 AWS 和 GCP 上。

在安装弹性云是很容易: 一个单一的点击创建一个 Elasticsearch 集群配置为你想要的大小,具有或不具有高可用性.
订阅功能始终安装, 所以你有自动保护和监视群集的能力.
一次点击 Kibana 可以在群集上启用, ，并且容易得到一些流行的插件.

一些弹性云功能只能与特定订阅使用. 欲了解更多信息，请参阅https://www.elastic.co/pricing/.

你可以[尝试一下 Elasticsearch 服务是免费的](https://www.elastic.co/cloud/elasticsearch-service/signup?baymax=docs-body&elektra=docs)。
欲了解更多信息，请参阅[入门弹性云](https://www.elastic.co/guide/en/cloud/current/ec-getting-started.html)。
