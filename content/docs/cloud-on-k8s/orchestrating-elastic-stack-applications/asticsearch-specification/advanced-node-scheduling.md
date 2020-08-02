---
title: "高级Elasticsearch节点调度"
linkTitle: "节点调度"
weight: 14
---

Elastic Cloud on Kubernetes (ECK) offers full control over Elasticsearch cluster nodes scheduling by combining Elasticsearch configuration with Kubernetes scheduling options:

Define Elasticsearch nodes roles
Pod affinity and anti-affinity
Availability zone and rack awareness
Hot-warm topologies
These features can be combined together, to deploy a production-grade Elasticsearch cluster.

Define Elasticsearch nodes rolesedit
You can configure Elasticsearch nodes with one or multiple roles. This allows you to describe an Elasticsearch cluster with 3 dedicated master nodes, for example:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
    # 3 dedicated master nodes
    - name: master
      count: 3
      config:
        node.master: true
        node.data: false
        node.ingest: false
        node.remote_cluster_client: false
    # 3 ingest-data nodes
    - name: ingest-data
      count: 3
      config:
        node.master: false
        node.data: true
        node.ingest: true
```

Affinity optionsedit
You can setup various affinity and anti-affinity options through the podTemplate section of the Elasticsearch resource specification.

A single Elasticsearch node per Kubernetes host (default)edit
To avoid scheduling several Elasticsearch nodes from the same cluster on the same host, use a podAntiAffinity rule based on the hostname and the cluster name label:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodesSets:
    - name: default
      count: 3
      podTemplate:
        spec:
          affinity:
            podAntiAffinity:
              preferredDuringSchedulingIgnoredDuringExecution:
                - weight: 100
                  podAffinityTerm:
                    labelSelector:
                      matchLabels:
                        elasticsearch.k8s.elastic.co/cluster-name: quickstart
                    topologyKey: kubernetes.io/hostname
```

This is ECK default behaviour if you don’t specify any affinity option.

To explicitly disable that behaviour, set an empty affinity object:

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
      podTemplate:
        spec:
          affinity: {}
```

The default affinity is using preferredDuringSchedulingIgnoredDuringExecution, which acts as best effort and won’t prevent an Elasticsearch node from being scheduled on a host if there are no other hosts available. Scheduling a 4-nodes Elasticsearch cluster on a 3-host Kubernetes cluster would then successfully schedule 2 Elasticsearch nodes on the same host. To enforce a strict single node per host, specify requiredDuringSchedulingIgnoredDuringExecution instead:

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
      podTemplate:
        spec:
          affinity:
            podAntiAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                - labelSelector:
                    matchLabels:
                      elasticsearch.k8s.elastic.co/cluster-name: quickstart
                  topologyKey: kubernetes.io/hostname
```

Local Persistent Volume constraintsedit
By default, volumes can be bound to a pod before the pod gets scheduled to a particular Kubernetes node. This can be a problem if the PersistentVolume can only be accessed from a particular host or set of hosts. Local persistent volumes are a good example: they are accessible from a single host. If the pod gets scheduled to a different host based on any affinity or anti-affinity rule, the volume may not be available.

To solve this problem, you can set the Volume Binding Mode of the StorageClass you are using. Make sure volumeBindingMode: WaitForFirstConsumer is set, especially if you are using local persistent volumes.

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: local-storage
provisioner: kubernetes.io/no-provisioner
volumeBindingMode: WaitForFirstConsumer
Node affinityedit
To restrict the scheduling to a particular set of Kubernetes nodes based on labels, use a NodeSelector. The following example schedules Elasticsearch pods on Kubernetes nodes tagged with both labels diskType: ssd and environment: production.

apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
  - name: default
    count: 3
    podTemplate:
      spec:
        nodeSelector:
          diskType: ssd
          environment: production
```

You can achieve the same (and more) with node affinity:

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
      podTemplate:
        spec:
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: environment
                        operator: In
                        values:
                          - e2e
                          - production
              preferredDuringSchedulingIgnoredDuringExecution:
                - weight: 1
                  preference:
                    matchExpressions:
                      - key: diskType
                        operator: In
                        values:
                          - ssd
```

This example restricts Elasticsearch nodes so they are only scheduled on Kubernetes hosts tagged with environment: e2e or environment: production. It favors nodes tagged with diskType: ssd.

Availability zone awarenessedit
By combining Elasticsearch shard allocation awareness with Kubernetes node affinity, you can setup an availability zone-aware Elasticsearch cluster:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
    - name: zone-a
      count: 1
      config:
        node.attr.zone: europe-west3-a
        cluster.routing.allocation.awareness.attributes: zone
      podTemplate:
        spec:
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: failure-domain.beta.kubernetes.io/zone
                        operator: In
                        values:
                          - europe-west3-a
    - name: zone-b
      count: 1
      config:
        node.attr.zone: europe-west3-b
        cluster.routing.allocation.awareness.attributes: zone
      podTemplate:
        spec:
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: failure-domain.beta.kubernetes.io/zone
                        operator: In
                        values:
                          - europe-west3-b
```

This example relies on:

Kubernetes nodes in each zone being labeled accordingly. failure-domain.beta.kubernetes.io/zone is standard, but any label can be used.
node affinity for each group of nodes set to match the Kubernetes nodes' zone.
Elasticsearch configured to allocate shards based on node attributes. Here we specified node.attr.zone, but any attribute name can be used. node.attr.rack_id is another common example.
Hot-warm topologiesedit
By combining Elasticsearch shard allocation awareness with Kubernetes node affinity, you can setup an Elasticsearch cluster with hot-warm topology:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: quickstart
spec:
  version: 7.8.1
  nodeSets:
    # hot nodes, with high CPU and fast IO
    - name: hot
      count: 3
      config:
        node.attr.data: hot
      podTemplate:
        spec:
          containers:
            - name: elasticsearch
              resources:
                limits:
                  memory: 16Gi
                  cpu: 4
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: beta.kubernetes.io/instance-type
                        operator: In
                        values:
                          - highio
      volumeClaimTemplates:
        - metadata:
            name: elasticsearch-data
          spec:
            accessModes:
              - ReadWriteOnce
            resources:
              requests:
                storage: 1Ti
            storageClassName: local-storage
    # warm nodes, with high storage
    - name: warm
      count: 3
      config:
        node.attr.data: warm
      podTemplate:
        spec:
          containers:
            - name: elasticsearch
              resources:
                limits:
                  memory: 16Gi
                  cpu: 2
          affinity:
            nodeAffinity:
              requiredDuringSchedulingIgnoredDuringExecution:
                nodeSelectorTerms:
                  - matchExpressions:
                      - key: beta.kubernetes.io/instance-type
                        operator: In
                        values:
                          - highstorage
      volumeClaimTemplates:
        - metadata:
            name: elasticsearch-data
          spec:
            accessModes:
              - ReadWriteOnce
            resources:
              requests:
                storage: 10Ti
            storageClassName: local-storage
```

In this example, we configure two groups of Elasticsearch nodes:

the first group has the data attribute set to hot. It is intended to run on hosts with high CPU resources and fast IO (SSD). Here we restrict pods to be scheduled on Kubernetes nodes labeled with beta.kubernetes.io/instance-type: highio (to adapt to your Kubernetes nodes' labels).
the second group has the data attribute set to warm. It is intended to run on hosts with larger but maybe slower storage. Pods are only able to be scheduled on nodes labeled with beta.kubernetes.io/instance-type: highstorage.
this example uses Local Persistent Volumes for both groups, but can be adapted to use high-performance volumes for hot Elasticsearch nodes and high-storage volumes for warm Elasticsearch nodes.

Finally, setup Index Lifecycle Management policies on your indices, optimizing for hot-warm architectures.
