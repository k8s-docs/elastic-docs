---
title: "Pod破坏预算"
linkTitle: ""
weight: 12
---

A Pod Disruption Budget allows limiting disruptions on an existing set of Pods while the Kubernetes cluster administrator manages Kubernetes nodes. Elasticsearch makes sure some indices don’t become unavailable.

ECK manages a default PDB per Elasticsearch resource. It allows one Elasticsearch Pod to be taken down, as long as the cluster has a green health. Single-node clusters are not considered highly available and can always be disrupted.

This default can be tweaked in the Elasticsearch specification:

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
  podDisruptionBudget:
    spec:
      minAvailable: 2
      selector:
        matchLabels:
          elasticsearch.k8s.elastic.co/cluster-name: quickstart
```

Note that maxUnavailable cannot be used with an arbitrary label selector, hence the usage of minAvailable in this example.

The default PDB can also be explicitly disabled:

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
  podDisruptionBudget: {}
```
