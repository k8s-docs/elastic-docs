---
title: "elasticsearch.k8s.elastic.co/v1beta1"
linkTitle: ""
weight: 1
---

Package v1beta1 contains API schema definitions for managing Elasticsearch resources.

Resource Types

Elasticsearch

## ChangeBudget

ChangeBudget defines the constraints to consider when applying changes to the Elasticsearch cluster.

Appears In:

UpdateStrategy
Field Description
maxUnavailable integer

MaxUnavailable is the maximum number of pods that can be unavailable (not ready) during the update due to circumstances under the control of the operator. Setting a negative value will disable this restriction. Defaults to 1 if not specified.

maxSurge integer

MaxSurge is the maximum number of new pods that can be created exceeding the original number of pods defined in the specification. MaxSurge is only taken into consideration when scaling up. Setting a negative value will disable the restriction. Defaults to unbounded if not specified.

## Elasticsearch

Elasticsearch represents an Elasticsearch resource in a Kubernetes cluster.

Field Description
apiVersion string

elasticsearch.k8s.elastic.co/v1beta1

kind string

Elasticsearch

metadata ObjectMeta

Refer to Kubernetes API documentation for fields of metadata.

spec ElasticsearchSpec

## ElasticsearchSpec

ElasticsearchSpec holds the specification of an Elasticsearch cluster.

Appears In:

Elasticsearch
Field Description
version string

Version of Elasticsearch.

image string

Image is the Elasticsearch Docker image to deploy.

http HTTPConfig

HTTP holds HTTP layer settings for Elasticsearch.

nodeSets NodeSet array

NodeSets allow specifying groups of Elasticsearch nodes sharing the same configuration and Pod templates.

updateStrategy UpdateStrategy

UpdateStrategy specifies how updates to the cluster should be performed.

podDisruptionBudget PodDisruptionBudgetTemplate

PodDisruptionBudget provides access to the default pod disruption budget for the Elasticsearch cluster. The default budget selects all cluster pods and sets maxUnavailable to 1. To disable, set PodDisruptionBudget to the empty value ({} in YAML).

secureSettings SecretSource

SecureSettings is a list of references to Kubernetes secrets containing sensitive configuration options for Elasticsearch.

## NodeSet

Appears In:

ElasticsearchSpec
Field Description
name string

Name of this set of nodes. Becomes a part of the Elasticsearch node.name setting.

config Config

Config holds the Elasticsearch configuration.

count integer

Count of Elasticsearch nodes to deploy.

podTemplate PodTemplateSpec

PodTemplate provides customisation options (labels, annotations, affinity rules, resource requests, and so on) for the Pods belonging to this NodeSet.

volumeClaimTemplates PersistentVolumeClaim array

VolumeClaimTemplates is a list of persistent volume claims to be used by each Pod in this NodeSet. Every claim in this list must have a matching volumeMount in one of the containers defined in the PodTemplate. Items defined here take precedence over any default claims added by the operator with the same name.

## UpdateStrategy

UpdateStrategy specifies how updates to the cluster should be performed.

Appears In:

ElasticsearchSpec
Field Description
changeBudget ChangeBudget

ChangeBudget defines the constraints to consider when applying changes to the Elasticsearch cluster.
