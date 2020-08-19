---
title: "配置"
linkTitle: ""
weight: 2
---

## 升级 Beat 规范

在 Beat YAML 规范里任何设置都可以被改变 , 包括版本升级.
ECK 检测这些变化 并确保所有 Beat 的 Pods 平滑滚动升级.
根据规范设置使用, ECK 将设置配置的输出部分, 执行 Kibana 仪表板安装, 重启 Beats 上的证书侧翻并设置了 Beats 密钥库.

## 自定义 Beat 配置

在 Beat 配置是通过配置元件提供:

```yaml
apiVersion: beat.k8s.elastic.co/v1beta1
kind: Beat
metadata:
  name: quickstart
spec:
  type: heartbeat
  version: 7.8.1
  elasticsearchRef:
    name: quickstart
  config:
    heartbeat.monitors:
      - type: tcp
        schedule: "@every 5s"
        hosts: ["quickstart-es-http.default.svc:9200"]
  deployment:
    podTemplate:
      spec:
        dnsPolicy: ClusterFirstWithHostNet
        securityContext:
          runAsUser: 0
```

另外, 它可以通过在 configRef 元素中指定一个秘密来提供:

```yaml
apiVersion: beat.k8s.elastic.co/v1beta1
kind: Beat
metadata:
  name: heartbeat-quickstart
spec:
  type: heartbeat
  version: 7.8.0
  elasticsearchRef:
    name: quickstart
  configRef:
    secretName: heartbeat-config
  deployment:
    podTemplate:
      spec:
        dnsPolicy: ClusterFirstWithHostNet
        securityContext:
          runAsUser: 0
---
apiVersion: v1
kind: Secret
metadata:
  name: heartbeat-config
stringData:
  beat.yml: |-
    heartbeat.monitors:
    - type: tcp
      schedule: '@every 5s'
      hosts: ["quickstart-es-http.default.svc:9200"]
```

有关 Beats 配置的详细信息, 看到了 Beats 文档.

## 部署 Beat

ECK 支持以下的 Beats 部署:

- Filebeat
- Metricbeat
- Heartbeat
- Auditbeat
- Packetbeat
- Journalbeat

您可以指定 Beat 部署 和 其版本通过类型和版本元素.
ECK 创造了 Elasticsearch 新用户 同 最小的一套合适的角色和权限以允许使用的所有 Beats 功能.

## 部署社区 Beat

ECK 支持任何社区 Beat 部署.
类型和版本规范元件必须提供. 此外:

1. 在本说明书中元素必须指向镜像被部署
2. 以下角色必须存在于 Elasticsearch:

   - 如果 elasticsearchRef 提供, 与 ECK`eck_beat_es_$type_role`名称的角色，必须存在, 其中`$type`是 Beat 类型. 例如, 部署 kafkabeat 当, 角色名称会 eck_beat_es_kafkabeat_role. 此角色必须拥有的权限要求的 Beat.
     详情请参见该 Elasticsearch 文档.
   - 如果 kibanaRef 提供, 然后, 如上, 一个名为`eck_beat_kibana_$type_role`角色必须存在的 与需要设置 Kibana 仪表板的权限.

另外, 创建 Elasticsearch 用户，包括凭据 在 Beats 配置为 Elasticsearch 输出, Kibana 设置或两者.
如果 elasticsearchRef 和 kibanaRef 也被定义, 建立连接时，ECK 将使用所提供的用户凭据.

## 设置 Kibana 仪表板

ECK 可以指示 Beats 设置封装例如仪表板 与 Beat.
要启用此, 设置 kibanaRef 元件在说明书指向 ECK-managed Kibana 部署:

```yaml
apiVersion: beat.k8s.elastic.co/v1beta1
kind: Beat
metadata:
  name: quickstart
spec:
  kibanaRef:
    name: quickstart
```

ECK 将创建 Elasticsearch 新用户 有一组合适的角色和权限最小 所需要的仪表盘设置.

## 为安全设置的秘密密钥存储

Beats 提供一个秘密密钥存储敏感设置 这在配置中提供必要, 例如密码.
这避免了在配置存储直接它们.

ECK 公开在说明书的 secureSettings 元素机构.
类似 Elasticsearch, 您可以使用 Kubernetes 秘密安全地提供的设置:

```yaml
apiVersion: beat.k8s.elastic.co/v1beta1
kind: Beat
metadata:
  name: quickstart
spec:
  secureSettings:
    - secretName: agent-name-secret
  config:
    name: ${AGENT_NAME_VAR}
---
apiVersion: v1
kind: Secret
metadata:
  name: agent-name-secret
stringData:
  AGENT_NAME_VAR: id_007
```

更多细节见 Beats 文档.

## 设置 Beat 输出

如果指定了 elasticsearchRef 元件, ECK 填充 Beat 配置的输出部.
ECK 创建具有适当的角色和权限的用户，并使用其凭据.
如果需要, 它还安装了 CA 证书 在所有 Beat `Pods`, 并重新`Pods`当此证书的变化.

输出可以被设置为任意值 由给定的 Beat 支持.
要使用它, 除去 elasticsearchRef 元件 从规格 和包括一个适当的输出配置 在配置或 configRef 元件.

```yaml
apiVersion: beat.k8s.elastic.co/v1beta1
kind: Beat
metadata:
  name: quickstart
spec:
  config:
    output.kafka:
      hosts: ["kafka1.default.svc:9092", "kafka2.default.svc:9092"]
      topic: "%{[fields.log_topic]}"
      partition.round_robin:
        reachable_only: false
      required_acks: 1
```

## 选择部署模型

根据不同的使用情况, Beats 可能需要部署为 Deployment 或 DaemonSet.
提供 podTemplate 元素 可选择按照`deployment`或 daemonSet 元件 在规范 选择给定的 Beat 应如何部署.

## Beats 的基于角色的访问控制

一些 Beats 功能 (如自动发现或 Kubernetes 模块 metricsets) 要求 Beat `Pods` 与 Kubernetes 交互的 API.
需要具体的权限，以允许该功能.
标准 Kubernetes RBAC 规则.
例如，允许自动发现:

```yaml
apiVersion: beat.k8s.elastic.co/v1beta1
kind: Beat
metadata:
  name: quickstart
spec:
  config:
    filebeat:
      autodiscover:
        providers:
          - host: ${HOSTNAME}
            type: kubernetes
            hints:
              enabled: true
              default_config:
                type: container
                paths:
                  - /var/log/containers/*${data.kubernetes.container.id}.log
  daemonSet:
    podTemplate:
      spec:
        serviceAccount: elastic-beat-filebeat-quickstart
        automountServiceAccountToken: true
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: elastic-beat-filebeat-quickstart
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: elastic-beat-autodiscover-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: elastic-beat-autodiscover
subjects:
  - kind: ServiceAccount
    name: elastic-beat-filebeat-quickstart
    namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: elastic-beat-autodiscover
rules:
  - apiGroups:
      - ""
    resources:
      - nodes
      - namespaces
      - events
      - pods
    verbs:
      - get
      - list
      - watch
```

## 在安全的集群部署 Beats

要在集群中部署 Beats 与`Pod`安全策略准入控制器启用, 或 OpenShift 集群, 您必须授予额外权限到服务帐户所用的 Beat `Pods`.
这些服务帐户必须绑定到一个角色或 ClusterRole 有使用许可 f 或所需`Pod`安全策略或安全环境约束.
不同的 Beats 及其功能可能 需要在他们的 PSP / SCC 设置不同的设置.
