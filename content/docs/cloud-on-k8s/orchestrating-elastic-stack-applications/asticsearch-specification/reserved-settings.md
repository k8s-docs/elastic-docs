---
title: "通过ECK管理设置"
linkTitle: "管理设置"
weight: 7
---

以下 Elasticsearch 设置通过 ECK 管理 :

- cluster.name
- discovery.zen.minimum_master_nodes [7.0]Deprecated in 7.0.
- cluster.initial_master_nodes [7.0]Added in 7.0.
- network.host
- network.publish_host
- path.data
- path.logs
- xpack.security.authc.reserved_realm.enabled
- xpack.security.enabled
- xpack.security.http.ssl.certificate
- xpack.security.http.ssl.enabled
- xpack.security.http.ssl.key
- xpack.security.transport.ssl.certificate
- xpack.security.transport.ssl.enabled
- xpack.security.transport.ssl.key
- xpack.security.transport.ssl.verification_mode

虽然 ECK 不会阻止你设定的任何设置自己, 我们强烈劝阻 这样做 和 我们不能提供支持 对于 任何用户提供 Elasticsearch 配置 这并使用其中的任何设置.
