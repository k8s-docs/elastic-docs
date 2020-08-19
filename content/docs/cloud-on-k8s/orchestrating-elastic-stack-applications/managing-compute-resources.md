---
title: "管理计算资源"
linkTitle: ""
weight: 2
---

To help the Kubernetes scheduler correctly place Pods in available Kubernetes nodes and ensure quality of service (QoS), it is recommended to specify the CPU and memory requirements for objects managed by the operator (Elasticsearch, Kibana or APM Server). In Kubernetes, requests defines the minimum amount of resources that must be available for a Pod to be scheduled; limits defines the maximum amount of resources that a Pod is allowed to consume. For more information about how Kubernetes uses these concepts, see: Managing Compute Resources for Containers.

Set compute resourcesedit
You can set compute resource constraints in the podTemplate of objects managed by the operator.

Set compute resources for Elasticsearchedit
For Elasticsearch objects, make sure to consider the heap size when you set resource requirements. A good rule of thumb is to size it to half the size of RAM allocated to the Pod.

To minimize disruption caused by Pod evictions due to resource contention, you should run Elasticsearch pods at the "Guaranteed" QoS level by setting both requests and limits to the same value.

Consider also that Kubernetes throttles containers exceeding the CPU limit defined in the limits section. Do not set this value too low or it would affect the performance of Elasticsearch, even if you have enough resources available in the Kubernetes cluster.

A known Kubernetes issue may lead to over-aggressive CPU limits throttling. If the host Linux Kernel does not include this CFS quota fix, you may want to:

not set any CPU limit in the Elasticsearch resource (Burstable QoS)
reduce the CFS quota period in kubelet configuration
disable CFS quotas in kubelet configuration
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
name: quickstart
spec:
version: 7.8.1
nodeSets:

- name: default
  count: 1
  podTemplate:
  spec:
  containers: - name: elasticsearch
  env: - name: ES_JAVA_OPTS
  value: -Xms2g -Xmx2g
  resources:
  requests:
  memory: 4Gi
  cpu: 0.5
  limits:
  memory: 4Gi
  cpu: 2
  Set compute resources for Kibana and APM Serveredit
  For Kibana or APM Server objects, the podTemplate can be configured as follows:

apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
name: quickstart
spec:
version: 7.8.1
podTemplate:
spec:
containers: - name: kibana
resources:
requests:
memory: 1Gi
cpu: 0.5
limits:
memory: 2Gi
cpu: 2
For the container name, you can use apm-server or kibana as appropriate.

Default behavioredit
If resources is not defined in the specification of an object, then the operator applies a default memory limit to ensure that pods have enough resources to start correctly. As the operator cannot make assumptions about the available CPU resources in the cluster, no CPU limits will be set — resulting in the pods having the "Burstable" QoS class. Check if this is acceptable for your use case and follow the instructions in Set compute resources to configure appropriate limits.

Table 1. Default limits applied by the operator

Type Requests Limits
APM Server

512Mi

512Mi

Elasticsearch

2Gi

2Gi

Kibana

1Gi

1Gi

If the Kubernetes cluster is configured with LimitRanges that enforce a minimum memory constraint, they could interfere with the operator defaults and cause object creation to fail.

For example, you might have a LimitRange that enforces a default and minimum memory limit on containers as follows:

apiVersion: v1
kind: LimitRange
metadata:
name: default-mem-per-container
spec:
limits:

- min:
  memory: "3Gi"
  defaultRequest:
  memory: "3Gi"
  type: Container
  With the above restriction in place, if you create an Elasticsearch object without defining the resources section, you will get the following error:

Cannot create pod elasticsearch-sample-es-ldbgj48c7r: pods "elasticsearch-sample-es-ldbgj48c7r" is forbidden: minimum memory usage per Container is 3Gi, but request is 2Gi
To avoid this, explicitly define the requests and limits mandated by your environment in the resource specification. It will prevent the operator from applying the built-in defaults.
