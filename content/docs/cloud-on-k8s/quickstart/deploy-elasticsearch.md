---
title: "部署Elasticsearch集群"
linkTitle: "部署ES"
weight: 2
---

套用一个简单的 [Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.8/getting-started.html) 集群规范, 一个 Elasticsearch 节点:

如果您 Kubernetes 群集任何 Kubernetes 节点没有至少 2GiB 的空闲内存, pod 将陷入等待状态.
请参阅[管理计算资源](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-managing-compute-resources.html)有关的资源需求，以及如何配置它们的详细信息。

```sh
cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
  - name: default
    count: 1
    config:
      node.master: true
      node.data: true
      node.ingest: true
      node.store.allow_mmap: false
EOF
```

操作者自动创建和管理 Kubernetes 资源来实现 Elasticsearch 群集的所期望的状态。
它可能需要几分钟，直到创建了所有的资源和集群已经可以使用。

设置 node.store.allow_mmap: false 具有性能影响并如在[虚拟存储器](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-virtual-memory.html)部分中所描述应调整为生产工作负载 .

## 监控集群健康和创建进度

在 Kubernetes 集群中获取当前 Elasticsearch 集群的概述 , 包括节点的健康，版本和数量:

```sh
kubectl get elasticsearch

NAME          HEALTH    NODES     VERSION   PHASE         AGE
quickstart    green     1         7.8.1     Ready         1m
```

当您创建集群, 没有`HEALTH`状态和`PHASE`是空的.
过了一会儿, 在`PHASE`转化成了就绪, 和`HEALTH`成为`green`.

你可以看到一个`Pod`是正在启动的过程:

```
kubectl get pods --selector='elasticsearch.k8s.elastic.co/cluster-name=quickstart'

NAME                      READY   STATUS    RESTARTS   AGE
quickstart-es-default-0   1/1     Running   0          79s
```

访问`Pod`日志:

```sh
kubectl logs -f quickstart-es-default-0
```

## 请求 Elasticsearch 访问

一个 ClusterIP 服务为您的集群自动创建:

```
kubectl get service quickstart-es-http

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
quickstart-es-http   ClusterIP   10.15.251.145   <none>        9200/TCP   34m
```

1. 获得证书.

   名为 elastic 默认用户被自动创建并带有存储在 Kubernetes secret 密码:

   ```sh
   PASSWORD=$(kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}')
   ```

2. 请求 Elasticsearch 端点.

   在 Kubernetes 集群内:

   ```sh
   curl -u "elastic:$PASSWORD" -k "https://quickstart-es-http:9200"
   ```

   从本地工作站, 在一个单独的终端使用下面的命令:

   ```sh
   kubectl port-forward service/quickstart-es-http 9200
   ```

   然后要求本地主机:

   ```sh
   curl -u "elastic:$PASSWORD" -k "https://localhost:9200"
   ```

> 运用 -k 参数禁用证书验证 不推荐 并应仅用于测试目的. 看: [设置自己的证书](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-tls-certificates.html#k8s-setting-up-your-own-certificate)

```json
{
  "name" : "quickstart-es-default-0",
  "cluster_name" : "quickstart",
  "cluster_uuid" : "XqWg0xIiRmmEBg4NMhnYPg",
  "version" : {...},
  "tagline" : "You Know, for Search"
}
```
