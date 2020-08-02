---
title: "限制跨名称空间的资源关联"
linkTitle: ""
weight: 3
---

This section describes how to restrict associations that can be created between resources managed by ECK.

When using the elasticsearchRef field to establish a connection to Elasticsearch from Kibana, APM Server, Enterprise Search, or Beats resources, by default the association is allowed as long as both resources are deployed to namespaces managed by that particular ECK instance. The association will succeed even if the user creating the association does not have access to one of the namespaces or the Elasticsearch resource.

The enforcement of access control rules for cross-namespace associations is disabled by default. Once enabled, it only enforces access control for resources deployed across two different namespaces. Associations between resources deployed in the same namespace are not affected.

Associations are allowed as long as the ServiceAccount used by the associated resource can execute HTTP GET requests against the referenced Elasticsearch object.

ECK automatically removes any associations that do not have the correct access rights. If you have existing associations, do not enable this feature without creating the required Roles and RoleBindings as described in the following sections.

To enable the restriction of cross-namespace associations, start the operator with the --enforce-rbac-on-refs flag.

Create a ClusterRole to allow HTTP GET requests to be run against Elasticsearch objects:

apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
name: elasticsearch-association
rules:

- apiGroups: - elasticsearch.k8s.elastic.co
  resources: - elasticsearches
  verbs: - get
  Create a ServiceAccount and a RoleBinding in the Elasticsearch namespace to allow any resource using the ServiceAccount to associate with the Elasticsearch cluster:

> kubectl create serviceaccount associated-resource-sa
> apiVersion: rbac.authorization.k8s.io/v1
> kind: RoleBinding
> metadata:
> name: allow-associated-resource-from-remote-namespace
> namespace: elasticsearch-ns
> roleRef:
> apiGroup: rbac.authorization.k8s.io
> kind: ClusterRole
> name: elasticsearch-association
> subjects:

- kind: ServiceAccount
  name: associated-resource-sa
  namespace: associated-resource-ns
  Set the serviceAccountName field in the associated resource to specify which ServiceAccount is used to create the association:

apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
name: associated-resource
namespace: associated-resource-ns
spec:
...
elasticsearchRef:
name: "elasticsearch-sample"
namespace: "elasticsearch-ns"

# Service account used by this resource to get access to an Elasticsearch cluster

serviceAccountName: associated-resource-sa
In the above example, associated-resource can be of any Kind that requires an association to be created, for example Kibana or ApmServer. You can find a complete example in the ECK GitHub repository.

If the serviceAccountName is not set, ECK uses the default service account assigned to the pod by the Service Account Admission Controller.

The associated resource associated-resource is now allowed to create an association with any Elasticsearch cluster in the namespace elasticsearch-ns.
