---
title: "循环自动生成凭证"
linkTitle: ""
weight: 3
description: >
  你可以通过简单地删除相应的秘密强制自动生成凭证与新值再生.
---

在部署弹性堆叠应用, 操作者产生一组凭证必不可少的用于该应用程序的操作.
这些生成的凭证的例子包括 Elasticsearch 默认弹性用户和令牌的 APM 服务器的安全性.
您可以通过运行以下命令列出命名空间中的所有自动生成凭证:

```sh
kubectl get secret -l eck.k8s.elastic.co/credentials=true
```

你可以通过简单地删除相应的秘密强制自动生成凭证与新值再生.
例如，修改密码，用于从快速启动例如弹性用户，执行以下命令:

```sh
kubectl delete secret quickstart-es-elastic-user
```

> 如果您使用的是自己的应用程序弹性用户凭据，他们将无法通过上述步骤后连接到 Elasticsearch 和 Kibana。不推荐用于生产用例弹性的用户凭据。始终创建自己的用户限制角色的访问 Elasticsearch.

您可以通过执行以下命令重新命名空间中的所有自动生成凭证:

```sh
kubectl delete secret -l eck.k8s.elastic.co/credentials=true
```

> 上面的命令重新生成命名空间中的所有弹性堆叠应用自动生成凭证.
