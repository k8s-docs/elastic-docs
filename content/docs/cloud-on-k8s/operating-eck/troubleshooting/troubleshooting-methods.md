---
title: "故障排除方法"
linkTitle: "故障排除"
weight: 2
---

最常见的问题可以通过以下说明来识别和解决:

- View the list of resources
- Describe failing resources
- Enable ECK debug logs
- View logs
- Exclude a resource from reconciliation
- Get Kubernetes events
- Exec into containers

如果您仍然无法按照上述说明后，找到解决问题的方法, 请求帮忙:

- ECK Discuss forums to ask any question
- Github issues for bugs and feature requests

## 查看资源列表

部署和管理的弹性堆栈, ECK 在命名空间(其中主要的资源部署)中创建多个资源 .

例如, 每个 Elasticsearch 节点和 Kibana 实例具有专用`Pod`.
检查运行`Pods`的状态, 与预期的情况下进行比较:

```sh
kubectl get pods

NAME                                 READY     STATUS    RESTARTS   AGE
elasticsearch-sample-es-66sv6dvt7g   0/1       Pending   0          3s
elasticsearch-sample-es-9xzzhmgd4h   1/1       Running   0          27m
elasticsearch-sample-es-lgphkv9p67   0/1       Pending   0          3s
kibana-sample-kb-5468b8685d-c7mdp    0/1       Running   0          4s
```

检查服务:

```sh
kubectl get services

NAME                           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
elasticsearch-sample-es-http   ClusterIP   10.19.248.93    <none>        9200/TCP   2d
kibana-sample-kb-http          ClusterIP   10.19.246.116   <none>        5601/TCP   3d
```

## 描述失败的资源

如果 Elasticsearch 节点启动不起来, 这可能是因为 Kubernetes 不能安排相关的`Pod`.

第一, 检查 StatefulSets 以查看是否副本的当前数量匹配副本的期望数量的.

```sh
kubectl get statefulset

NAME                              DESIRED   CURRENT   AGE
elasticsearch-sample-es-default   1         1         4s
```

然后, 检查`Pod`状态. 如果`Pod`失败，几秒钟后达到运行状态, 某些东西是防止它被安排或启动:

```sh
kubectl get pods -l elasticsearch.k8s.elastic.co/statefulset-name=elasticsearch-sample-es-default

NAME                                 READY     STATUS    RESTARTS   AGE
elasticsearch-sample-es-66sv6dvt7g   0/1       Pending   0          3s
elasticsearch-sample-es-9xzzhmgd4h   1/1       Running   0          42s
elasticsearch-sample-es-lgphkv9p67   0/1       Pending   0          3s
kibana-sample-kb-5468b8685d-c7mdp    0/1       Running   0          4s
```

Pod elasticsearch-sample-es-lgphkv9p67 未计划.运行此命令，以获得更多的见解:

```sh
kubectl describe pod elasticsearch-sample-es-lgphkv9p67

(...)
Events:
  Type     Reason             Age               From                Message
  ----     ------             ----              ----                -------
  Warning  FailedScheduling   1m (x6 over 1m)   default-scheduler   pod has unbound immediate PersistentVolumeClaims (repeated 2 times)
  Warning  FailedScheduling   1m (x6 over 1m)   default-scheduler   pod has unbound immediate PersistentVolumeClaims
  Warning  FailedScheduling   1m (x11 over 1m)  default-scheduler   0/3 nodes are available: 1 node(s) had no available volume zone, 2 Insufficient memory.
  Normal   NotTriggerScaleUp  4s (x11 over 1m)  cluster-autoscaler  pod didn't trigger scale-up (it wouldn't fit if a new node is added)
```

如果你看到一个(unbound persistent volume claims (PVCs))错误,
这意味着目前还没有一个持久卷能够满足所述要求.
如果您正在使用自动配置存储 (例如亚马逊 EBS 佣金), 有时存储提供商可以花几分钟时间来准备卷, 所以这可能在几分钟内自行解决.
您还可以通过运行`kubectl describe persistentvolumeclaims`看到 PVC 的事件检查状态。

## 启用 ECK 调试日志

为了在运营商启动`DEBUG`级别日志 , 编辑`elastic-operator` StatefulSet 并设置 --log-verbosity 标志为 1 如下图所示.
一旦你的改变被保存, 运营商将由 StatefulSet 控制器自动重新启动 以应用新设置.

```sh
kubectl edit statefulset.apps -n elastic-system elastic-operator
```

改变 args 数组如下:

```yaml
spec:
  containers:
    - args:
        - manager
        - --log-verbosity=1
```

## 查看日志

### 查看 Elasticsearch 日志

每个 Elasticsearch 节点名称映射到对应的`Pod`名.
为了得到一个特定 Elasticsearch 节点的日志, 刚取`Pod`日志:

```sh
kubectl logs -f elasticsearch-sample-es-lgphkv9p67

(...)
{"type": "server", "timestamp": "2019-07-22T08:48:10,859+0000", "level": "INFO", "component": "o.e.c.s.ClusterApplierService", "cluster.name": "elasticsearch-sample", "node.name": "elasticsearch-sample-es-lgphkv9p67", "cluster.uuid": "cX9uCx3uQrej9hMLGPhV0g", "node.id": "R_OcheBlRGeqme1IZzE4_Q", "message": "added {{elasticsearch-sample-es-kqz4jmvj9p}{UGy5IX0UQcaKlztAoh4sLA}{3o_EUuZvRKW7R1C8b1zzzg}{10.16.2.232}{10.16.2.232:9300}{ml.machine_memory=27395555328, ml.max_open_jobs=20, xpack.installed=true},{elasticsearch-sample-es-stzz78k64p}{Sh_AzQcxRzeuIoOQWgru1w}{cwPoTFNnRAWtqsXWQtWbGA}{10.16.2.233}{10.16.2.233:9300}{ml.machine_memory=27395555328, ml.max_open_jobs=20, xpack.installed=true},}, term: 1, version: 164, reason: ApplyCommitRequest{term=1, version=164, sourceNode={elasticsearch-sample-es-9xzzhmgd4h}{tAi_bCPcSaO1OkLap4wmhQ}{E6VcWWWtSB2oo-2zmj9DMQ}{10.16.1.150}{10.16.1.150:9300}{ml.machine_memory=27395555328, ml.max_open_jobs=20, xpack.installed=true}}" }
{"type": "server", "timestamp": "2019-07-22T08:48:22,224+0000", "level": "INFO", "component": "o.e.c.s.ClusterApplierService", "cluster.name": "elasticsearch-sample", "node.name": "elasticsearch-sample-es-lgphkv9p67", "cluster.uuid": "cX9uCx3uQrej9hMLGPhV0g", "node.id": "R_OcheBlRGeqme1IZzE4_Q", "message": "added {{elasticsearch-sample-es-fn9wvxw6sh}{_tbAciHTStaAlUO6GtD9LA}{1g7_qsXwR0qjjfom05VwMA}{10.16.1.154}{10.16.1.154:9300}{ml.machine_memory=27395555328, ml.max_open_jobs=20, xpack.installed=true},}, term: 1, version: 169, reason: ApplyCommitRequest{term=1, version=169, sourceNode={elasticsearch-sample-es-9xzzhmgd4h}{tAi_bCPcSaO1OkLap4wmhQ}{E6VcWWWtSB2oo-2zmj9DMQ}{10.16.1.150}{10.16.1.150:9300}{ml.machine_memory=27395555328, ml.max_open_jobs=20, xpack.installed=true}}" }
```

您可以运行 Kibana 和 APM 服务器相同的命令.

### 查看初始容器日志

一个 Elasticsearch`Pod`运行一些初始化容器准备主要 Elasticsearch 容器的文件系统.
在某些情况下, 该`Pod`可能无法运行 (Status: Error or Status: CrashloopBackOff) 因为在 init 容器中的一个无法运行.
看看起始容器的状态和日志，以获得更多的细节.

### 查看 ECK 日志

由于 ECK 运营商仅仅是一个在集群 Kubernetes 里标准的`Pod`运行 , 你会为任何其他`Pod`你可以获取其日志:

```sh
kubectl -n elastic-system logs -f statefulset.apps/elastic-operator
```

运营商不断试图调和 Kubernetes 资源以匹配期望的状态.
与 INFO 级别日志提供一些见解关于到底是怎么回事.
以 ERROR 级别的日志预示着什么不会如预期.

由于乐观锁定, 你可以得到错误报告发生冲突，同时更新资源.
你可以忽略它们, 作为更新下一个和解的尝试经历, 这将几乎立即发生.

## 排除和解资源

为了进行调试, 您可能希望暂时阻止 ECK 从修改属于特定弹性堆栈资源 Kubernetes 资源.
去做这个, 注释弹性对象 为 eck.k8s.elastic.co/managed=false.
这个注解可以添加到任何下列类型的对象:

- Elasticsearch
- Kibana
- ApmServer

```yaml
metadata:
  annotations:
    eck.k8s.elastic.co/managed: "false"
```

或者在一行:

```sh
kubectl annotate elasticsearch quickstart --overwrite eck.k8s.elastic.co/managed=false
```

## 获取 Kubernetes 事件

ECK 会发出事件时:

- 重要的操作执行(例: 新 Elasticsearch`Pod`创建)
- 什么是错的，用户必须通知

取 Kubernetes 事件:

```sh
kubectl get events

(...)
28s       25m       58        elasticsearch-sample-es-p45nrjch29.15b3ae4cc4f7c00d   Pod                             Warning   FailedScheduling    default-scheduler                                         0/3 nodes are available: 1 node(s) had no available volume zone, 2 Insufficient memory.
28s       25m       52        elasticsearch-sample-es-wxpnzfhqbt.15b3ae4d86bc269f   Pod                             Warning   FailedScheduling    default-scheduler                                         0/3 nodes are available: 1 node(s) had no available volume zone, 2 Insufficient memory.
```

您可以过滤事件只显示那些与特定的 Elasticsearch 集群:

```sh
kubectl get event --namespace default --field-selector involvedObject.name=elasticsearch-sample

LAST SEEN   FIRST SEEN   COUNT     NAME                                    KIND            SUBOBJECT   TYPE      REASON    SOURCE                     MESSAGE
30m         30m          1         elasticsearch-sample.15b3ae303baa93c0   Elasticsearch               Normal    Created   elasticsearch-controller   Created pod elasticsearch-sample-es-4q7q2k8cl7
30m         30m          1         elasticsearch-sample.15b3ae303bab4f40   Elasticsearch               Normal    Created   elasticsearch-controller   Created pod elasticsearch-sample-es-jg7dsfkcp8
30m         30m          1         elasticsearch-sample.15b3ae303babdfc8   Elasticsearch               Normal    Created   elasticsearch-controller   Created pod elasticsearch-sample-es-xrxsp54jd5
```

您可以 Kibana 和 APM 服务器设置过滤器过.
请注意，对于 Kubernetes 事件缺省 TTL 为 1h, 所以，除非您的群集设置已被修改，你不会看到超过 1h 以前的事件.

## 调整持久卷

若要增加或减少磁盘的大小, 你不能直接改变 volumeClaimTemplate 的大小.
这是因为 StatefulSets 不允许修改 volumeClaimTemplates.
要解决此, 您可以创建新的尺寸新的节点集, 并从 Elasticsearch 规范删除旧的节点集.
例如, 只是改变节点集的名称和尺寸在现有 Elasticsearch 规范.
ECK 将自动开始将数据迁移到新的节点集，并删除旧的，当它被完全耗尽.
详情请参阅[StatefulSets 编排](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-orchestration.html#k8s-statefulsets)文档.

对于一个具体的例子，假设你开始使用此:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
    - name: default
      count: 3
      config:
        node.master: true
        node.data: true
        node.ingest: true
        node.store.allow_mmap: false
      volumeClaimTemplates:
        - metadata:
            name: elasticsearch-data
          spec:
            accessModes:
              - ReadWriteOnce
            resources:
              requests:
                storage: 5Gi
            storageClassName: standard
```

并希望将其提高到存储的 10Gi。 您可以更改节点集名称和卷的大小，像这样:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
    - name: default-10gi
      count: 3
      config:
        node.master: true
        node.data: true
        node.ingest: true
        node.store.allow_mmap: false
      volumeClaimTemplates:
        - metadata:
            name: elasticsearch-data
          spec:
            accessModes:
              - ReadWriteOnce
            resources:
              requests:
                storage: 10Gi
            storageClassName: standard
```

和 ECK 会自动创建一个新的 StatefulSet 并开始数据迁移到它.

## Exec 到容器

要解决文件系统, 配置或网络问题, 可以在 Elasticsearch 容器里直接运行 shell 命令 . 你可以用 kubectl 做到这一点:

```sh
kubectl exec -ti elasticsearch-sample-es-p45nrjch29 bash
```

这也可以为 Kibana 和 APM 服务器完成.
