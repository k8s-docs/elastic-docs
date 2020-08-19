---
title: "kibana.k8s.elastic.co/v1"
linkTitle: ""
weight: 1
---

Package v1 contains API schema definitions for managing Kibana resources.

Resource Types

Kibana

## Kibana

Kibana represents a Kibana resource in a Kubernetes cluster.

Field Description
apiVersion string

kibana.k8s.elastic.co/v1

kind string

Kibana

metadata ObjectMeta

Refer to Kubernetes API documentation for fields of metadata.

spec KibanaSpec

## KibanaSpec

KibanaSpec holds the specification of a Kibana instance.

Appears In:

Kibana
Field Description
version string

Version of Kibana.

image string

Image is the Kibana Docker image to deploy.

count integer

Count of Kibana instances to deploy.

elasticsearchRef ObjectSelector

ElasticsearchRef is a reference to an Elasticsearch cluster running in the same Kubernetes cluster.

config Config

Config holds the Kibana configuration. See: https://www.elastic.co/guide/en/kibana/current/settings.html

http HTTPConfig

HTTP holds the HTTP layer configuration for Kibana.

podTemplate PodTemplateSpec

PodTemplate provides customisation options (labels, annotations, affinity rules, resource requests, and so on) for the Kibana pods

secureSettings SecretSource

SecureSettings is a list of references to Kubernetes secrets containing sensitive configuration options for Kibana.

serviceAccountName string

ServiceAccountName is used to check access from the current resource to a resource (eg. Elasticsearch) in a different namespace. Can only be used if ECK is enforcing RBAC on references.
