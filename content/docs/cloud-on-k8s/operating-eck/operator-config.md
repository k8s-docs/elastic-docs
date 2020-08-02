---
title: "配置ECK"
linkTitle: ""
weight: 1
description: ECK可以使用任一命令行标志或环境变量来配置。
---

| Flag                      | 默认值                                       | 描述                                                                                                       |
| ------------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| auto-port-forward         | false                                        | 启用自动端口转发，以允许运行在集群外部的操作者。 对于开发利用 仅作为它在临时端口暴露 K8S 资源到 localhost. |
| ca-cert-rotate-before     | 24h                                          | Duration representing how long before expiration CA certificates should be re-issued.                      |
| ca-cert-validity          | 8760h                                        | Duration representing the validity period of a generated CA certificate.                                   |
| cert-rotate-before        | 24h                                          | Duration representing how long before expiration TLS certificates should be re-issued.                     |
| cert-validity             | 8760h                                        | Duration representing the validity period of a generated TLS certificate.                                  |
| container-registry        | docker.elastic.co                            | Container registry to use for pulling Elastic Stack container images.                                      |
| debug-http-listen         | localhost:6060                               | Listen address for the debug HTTP server. Only available in development mode.                              |
| development               | false                                        | Enable development mode. Only available as a CLI flag.                                                     |
| enable-tracing            | false                                        | 在操作过程中启用 APM 跟踪. 使用环境变量来配置 APM 服务器 URL，证书等. 查看详情[APM 转到代理参考] .         |
| enable-webhook            | false                                        | Enables a validating webhook server in the operator process.                                               |
| enforce-rbac-on-refs      | false                                        | Enables restrictions on cross-namespace resource association through RBAC.                                 |
| log-verbosity             | 0                                            | Verbosity level of logs. -2=Error, -1=Warn, 0=Info, 0 and above=Debug.                                     |
| manage-webhook-certs      | true                                         | Enables automatic webhook certificate management.                                                          |
| max-concurrent-reconciles | 3                                            | 每个控制器并发调和的最大数量 (Elasticsearch, Kibana, APM Server). 影响经营者的能力流程中同时改变.          |
| metrics-port              | 0                                            | Prometheus metrics port. Set to 0 to disable the metrics endpoint.                                         |
| namespaces                | ""                                           | 命名空间在此操作应该管理资源. 接受多个逗号分隔值. 默认为空，如果未指定或所有命名空间.                      |
| operator-namespace        | ""                                           | Namespace the operator runs in. Required.                                                                  |
| webhook-pods-label        | ""                                           | Label used to select pods running the webhook server.                                                      |
| webhook-secret            | ""                                           | K8s secret mounted into the path designated by webhook-cert-dir to be used for webhook certificates.       |
| webhook-cert-dir          | "{TempDir}/k8s-webhook-server/serving-certs" | Path to the directory that contains the webhook server key and certificate.                                |

Unless noted otherwise, environment variables can be used instead of flags to configure the operator as well. Simply convert the flag name to upper case and replace any dashes (-) with underscores (\_). For example, the log-verbosity flag can be set by an environment variable named LOG_VERBOSITY.

Duration values should be specified as numeric values suffixed by the time unit. For example, a duration of 10 hours should be specified as 10h. Acceptable time unit suffixes are:

| Suffix | Unit         |
| ------ | ------------ |
| ms     | Milliseconds |
| s      | Seconds      |
| m      | Minutes      |
| h      | Hours        |

Edit the elastic-operator StatefulSet to change any of the flag values. Enable ECK debug logs illustrates how to change the log level of the operator using this method.
