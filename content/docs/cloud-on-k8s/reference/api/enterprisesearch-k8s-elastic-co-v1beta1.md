---
title: "enterprisesearch.k8s.elastic.co/v1beta1"
linkTitle: ""
weight: 1
---

Package v1beta1 contains API schema definitions for managing Enterprise Search resources.

Resource Types

EnterpriseSearch

## EnterpriseSearch

EnterpriseSearch is a Kubernetes CRD to represent Enterprise Search.

Field Description
apiVersion string

enterprisesearch.k8s.elastic.co/v1beta1

kind string

EnterpriseSearch

metadata ObjectMeta

Refer to Kubernetes API documentation for fields of metadata.

spec EnterpriseSearchSpec

## EnterpriseSearchSpec

EnterpriseSearchSpec holds the specification of an Enterprise Search resource.

Appears In:

EnterpriseSearch
Field Description
version string

Version of Enterprise Search.

image string

Image is the Enterprise Search Docker image to deploy.

count integer

Count of Enterprise Search instances to deploy.

config Config

Config holds the Enterprise Search configuration.

configRef ConfigSource

ConfigRef contains a reference to an existing Kubernetes Secret holding the Enterprise Search configuration. Configuration settings are merged and have precedence over settings specified in config.

http HTTPConfig

HTTP holds the HTTP layer configuration for Enterprise Search resource.

elasticsearchRef ObjectSelector

ElasticsearchRef is a reference to the Elasticsearch cluster running in the same Kubernetes cluster.

podTemplate PodTemplateSpec

PodTemplate provides customisation options (labels, annotations, affinity rules, resource requests, and so on) for the Enterprise Search pods.

serviceAccountName string

ServiceAccountName is used to check access from the current resource to a resource (eg. Elasticsearch) in a different namespace. Can only be used if ECK is enforcing RBAC on references.
