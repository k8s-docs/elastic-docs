---
title: "创建自定义镜像"
linkTitle: ""
weight: 1
---

您可以创建自己的自定义 Elasticsearch 或 Kibana 镜像 而不是使用所述基本镜像 通过`Elastic`提供.
你可能要在镜像中这样做是为了预加载插件 而不是通过起始容器进行安装 每次`pod`开始.
去做这个, 你必须正确地使用官方的镜像为基准，为它的功能.
例如, 如果你想创建一个谷歌云存储库插件的 Elasticsearch 7.8.1 镜像, 你可以做以下:

创建包含 Dockerfile:

```docker
FROM docker.elastic.co/elasticsearch/elasticsearch:7.8.1
RUN bin/elasticsearch-plugin install --batch repository-gcs
```

构建镜像 同:

```sh
docker build --tag elasticsearch-gcs:7.8.1
```

有你的镜像的各种托管选项。
如果你使用谷歌 Kubernetes 引擎, 它会自动配置为使用谷歌容器注册 (浏览此处获取更多信息).
使用镜像, 那么你就可以推到注册表 同:

```sh
docker tag elasticsearch-gcs:7.8.1 gcr.io/$PROJECT-ID/elasticsearch-gcs:7.8.1
docker push gcr.io/$PROJECT-ID/elasticsearch-gcs:7.8.1
```

配置您的 Elasticsearch 规范使用新推镜像, 例如:

```yaml
spec:
  version: 7.8.1
  image: gcr.io/$PROJECT-ID/elasticsearch-gcs:7.8.1
```

提供了正确的版本总是需要基于版本字段有关的 API，并提供给它的功能 ECK 原因.

该步骤是 Azure Kubernetes 服务和 AWS 弹性容器注册相似.

欲了解更多信息，您可以查看以下引用:

- 在创建自定义镜像 Elasticsearch DOC
- 谷歌容器注册表文件
- Azure 容器注册表文件
- 亚马逊弹性容器注册文档
- OpenShift 容器平台注册表文件

## 覆盖缺省容器注册表

当创建 Elasticsearch，Kibana，或 APM 服务器资源, 操作员默认使用容器图象 从 docker.elastic.co 注册表拉.
如果你是在一个环境中，外部网络访问限制, 您可以配置操作使用不同的默认容器注册表 通过启动运营 与 --container-registry command-line 旗. (请参阅配置 ECK 有关如何使用命令行标志和环境变量来配置操作的更多信息).

T 他运营商期望容器镜像位于默认容器注册表特定路径.
确保你的容器镜像存储在正确的道路，并与堆栈版本号正确标记.
例如, 如果您的私人注册表 my.registry 你要部署的组件 从堆栈版本 7.8.1, 以下镜像路径应该存在:

- `my.registry/elasticsearch/elasticsearch:7.8.1`
- `my.registry/kibana/kibana:7.8.1`
- `my.registry/apm/apm-server:7.8.1`
