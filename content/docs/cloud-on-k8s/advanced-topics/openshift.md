---
title: "OpenShift上部署ECK"
linkTitle: ""
weight: 1
---

This page shows how to run ECK on OpenShift.

Before you begin
Deploy the operator
Deploy an Elasticsearch instance with a route
Deploy a Kibana instance with a route
Deploy an APM Server instance with a route
Only Elasticsearch and Kibana are compatible with the restricted Security Context Constraint. To run the APM Server on OpenShift you must allow the Pod to run with the anyuid SCC as described in Deploy an APM Server instance with a route

Before you beginedit
To run the instructions on this page, you must be a system:admin user or a user with the privileges to create Projects, CRDs, and RBAC resources at the cluster level.
Set virtual memory settings on the Kubernetes nodes.

Before deploying an Elasticsearch cluster with ECK, make sure that the Kubernetes nodes in your cluster have the correct vm.max_map_count sysctl setting applied. By default, Pods created by ECK are likely to run with the restricted Security Context Constraint (SCC) which restricts privileged access required to change this setting in the underlying Kubernetes nodes.

Alternatively, you can opt for setting node.store.allow_mmap: false at the Elasticsearch node configuration level, but note that this has performance implications and is not recommended for production workloads.

For more information, see: Virtual memory.
