---
title: "虚拟内存"
linkTitle: ""
weight: 6
---

默认, Elasticsearch 使用存储器映射（MMAP）以有效地访问索引.
平时, d 对 Linux 发行版的虚拟地址空间的默认值 过低的 Elasticsearch 到正常工作, 这可能导致存储器外的异常.
这就是为什么快速入门示例禁用的 mmap 通过 node.store.allow_mmap: false 设置.
对于生产工作负载, 强烈建议增加内核设置 vm.max_map_count 至 262144 然后离开 node.store.allow_mmap 未设置.

内核设置 vm.max_map_count=262144 可以直接在主机上设置 或通过专用起始容器, 它必须是特权.
要添加起始容器 改变主机内核设置 在你 Elasticsearch pod 启动之前, 你可以用下面的例子 Elasticsearch 规范:

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
    count: 3
    config:
      node.master: true
      node.data: true
      node.ingest: true
    podTemplate:
      spec:
        initContainers:
        - name: sysctl
          securityContext:
            privileged: true
          command: ['sh', '-c', 'sysctl -w vm.max_map_count=262144']
EOF
```

注意这需要运行特权容器的能力, 这很可能并非如此 在许多安全集群上.

欲了解更多信息，请参阅虚拟内存 Elasticsearch 文档.

可选, 您可以选择不同类型的文件系统实现对存储.
对于可能的选择, 查看店内模块文件.

```yaml
spec:
  nodeSets:
    - name: default
      count: 3
      config:
        index.store.type: niofs
```
