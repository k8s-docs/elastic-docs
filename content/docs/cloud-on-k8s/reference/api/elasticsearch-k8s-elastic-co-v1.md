---
title: "elasticsearch.k8s.elastic.co/v1"
linkTitle: ""
weight: 1
---

Package v1 contains API schema definitions for managing Elasticsearch resources.

Resource Types

Elasticsearch

## Auth

Auth contains user authentication and authorization security settings for Elasticsearch.

Appears In:

ElasticsearchSpec
Field Description
roles RoleSource array

Roles to propagate to the Elasticsearch cluster.

fileRealm FileRealmSource array

FileRealm to propagate to the Elasticsearch cluster.

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

elasticsearch.k8s.elastic.co/v1

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

transport TransportConfig

Transport holds transport layer settings for Elasticsearch.

nodeSets NodeSet array

NodeSets allow specifying groups of Elasticsearch nodes sharing the same configuration and Pod templates.

updateStrategy UpdateStrategy

UpdateStrategy specifies how updates to the cluster should be performed.

podDisruptionBudget PodDisruptionBudgetTemplate

PodDisruptionBudget provides access to the default pod disruption budget for the Elasticsearch cluster. The default budget selects all cluster pods and sets maxUnavailable to 1. To disable, set PodDisruptionBudget to the empty value ({} in YAML).

auth Auth

Auth contains user authentication and authorization security settings for Elasticsearch.

secureSettings SecretSource

SecureSettings is a list of references to Kubernetes secrets containing sensitive configuration options for Elasticsearch.

serviceAccountName string

ServiceAccountName is used to check access from the current resource to a resource (eg. a remote Elasticsearch cluster) in a different namespace. Can only be used if ECK is enforcing RBAC on references.

remoteClusters RemoteCluster array

RemoteClusters enables you to establish uni-directional connections to a remote Elasticsearch cluster.

## FileRealmSource

Appears In:

Auth
Field Description
SecretRef SecretRef

SecretName references a Kubernetes secret in the same namespace as the Elasticsearch resource. Multiple users and their roles mapping can be specified in a Kubernetes secret. The secret should contain 2 entries: - users: contain all users and the hash of their password (https://www.elastic.co/guide/en/elasticsearch/reference/current/security-settings.html#password-hashing-algorithms) - users_roles: contain the role to users mapping The format of those 2 entries must correspond to the expected file realm format, as specified in Elasticsearch documentation: https://www.elastic.co/guide/en/elasticsearch/reference/7.5/file-realm.html#file-realm-configuration. Example: --- # File realm in ES format (from the CLI or manually assembled) kind: Secret apiVersion: v1 metadata: name: my-filerealm stringData: users:

rdeniro:$2a$10$BBJ/ILiyJ1eBTYoRKxkqbuDEdYECplvxnqQ47uiowE7yGqvCEgj9W alpacino:$2a$10$cNwHnElYiMYZ/T3K4PvzGeJ1KbpXZp2PfoQD.gfaVdImnHOwIuBKS jacknich:{PBKDF2}50000$z1CLJt0MEFjkIK5iEfgvfnA6xq7lF25uasspsTKSo5Q=$XxCVLbaKDimOdyWgLCLJiyoiWpA/XDMe/xtVgn1r5Sg= users_roles:
admin:rdeniro power_user:alpacino,jacknich user:jacknich ---

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

## RemoteCluster

Appears In:

ElasticsearchSpec
Field Description
name string

Name is the name of the remote cluster as it is set in the Elasticsearch settings. The name is expected to be unique for each remote clusters.

elasticsearchRef ObjectSelector

ElasticsearchRef is a reference to an Elasticsearch cluster running within the same k8s cluster.

## RoleSource

Appears In:

Auth
Field Description
SecretRef SecretRef

SecretName references a Kubernetes secret in the same namespace as the Elasticsearch resource. Multiple roles can be specified in a Kubernetes secret, under a single "roles.yml" entry. The secret value must match the expected file-based specification as described in https://www.elastic.co/guide/en/elasticsearch/reference/current/defining-roles.html#roles-management-file. Example: --- kind: Secret apiVersion: v1 metadata: name: my-roles stringData: roles.yml:

## TransportConfig

TransportConfig holds the transport layer settings for Elasticsearch.

Appears In:

ElasticsearchSpec
Field Description
service ServiceTemplate

Service defines the template for the associated Kubernetes Service object.

## UpdateStrategy

UpdateStrategy specifies how updates to the cluster should be performed.

Appears In:

ElasticsearchSpec
Field Description
changeBudget ChangeBudget

ChangeBudget defines the constraints to consider when applying changes to the Elasticsearch cluster.
