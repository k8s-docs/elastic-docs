---
title: "配置ECK"
linkTitle: ""
weight: 1
description: ECK可以使用任一命令行标志或环境变量来配置。
---

| Flag                      | 默认值                                       | 描述                                                                                                       |
| ------------------------- | -------------------------------------------- | ---------------------------------------------------------------------------------------------------------- |
| auto-port-forward         | false                                        | 启用自动端口转发，以允许运行在集群外部的操作者。 对于开发利用 仅作为它在临时端口暴露 K8S 资源到 localhost. |
| ca-cert-rotate-before     | 24h                                          | 持续时间 代表 多久到期前 CA 证书应重发.                                                                    |
| ca-cert-validity          | 8760h                                        | 持续时间 代表 所产生的 CA 证书的有效期.                                                                    |
| cert-rotate-before        | 24h                                          | 持续时间 代表 多久到期前 TLS 证书应重发.                                                                   |
| cert-validity             | 8760h                                        | 持续时间 代表 所生成的 TLS 证书的有效期.                                                                   |
| container-registry        | docker.elastic.co                            | 容器注册表使用拉 Elastic Stack 容器镜像.                                                                   |
| debug-http-listen         | localhost:6060                               | 为调试 HTTP 服务器监听地址. 只有在开发模式下可用.                                                          |
| development               | false                                        | 启用开发模式. 仅提供 CLI 标志.                                                                             |
| enable-tracing            | false                                        | 在操作过程中启用 APM 跟踪. 使用环境变量来配置 APM 服务器 URL，证书等. 查看详情[APM 转到代理参考] .         |
| enable-webhook            | false                                        | 允许在操作过程中验证`webhook`服务器。                                                                      |
| enforce-rbac-on-refs      | false                                        | 启用通过 RBAC 跨命名空间资源关联限制.                                                                      |
| log-verbosity             | 0                                            | 日志的详细级别. -2=Error, -1=Warn, 0=Info, 0 and above=Debug.                                              |
| manage-webhook-certs      | true                                         | 启用自动`webhook`证书管理.                                                                                 |
| max-concurrent-reconciles | 3                                            | 每个控制器并发调和的最大数量 (Elasticsearch, Kibana, APM Server). 影响经营者的能力流程中同时改变.          |
| metrics-port              | 0                                            | `Prometheus`指标端口. 设置为 0 禁用指标端点.                                                               |
| namespaces                | ""                                           | 命名空间在此操作应该管理资源. 接受多个逗号分隔值. 默认为空，如果未指定或所有命名空间.                      |
| operator-namespace        | ""                                           | 命名空间中的运营商运行. 需要.                                                                              |
| webhook-pods-label        | ""                                           | 标签用于选择`pods` 运行`webhook`服务器.                                                                    |
| webhook-secret            | ""                                           | K8S 秘密安装到指定的路径 通过 webhook-cert-dir 用于`webhook`证书.                                          |
| webhook-cert-dir          | "{TempDir}/k8s-webhook-server/serving-certs" | 目录路径 包含`webhook`服务器密钥和证书.                                                                    |

除非另有说明, 环境变量可以用来代替标志配置运营商以及. 只需将标志名称转换为大写并更换破折号 (-) 用下划线 (\_).
例如,log-verbosity 标志可以由名为 LOG_VERBOSITY 的环境变量设置.

持续时间值应被指定为通过所述时间单元的后缀数字值.
例如, 10 小时的持续时间应指定为 10H.
可接受的时间单位后缀:

| 后缀 | 单位 |
| ---- | ---- |
| ms   | 毫秒 |
| s    | 秒   |
| m    | 分钟 |
| h    | 小时 |

编辑 elastic-operator StatefulSet 更改任何标志值的.
启用 ECK 调试日志 说明了如何使用这种方法来改变经营者的日志级别.
