---
title: "升级ECK"
linkTitle: ""
weight: 6
---

## Upgrade to ECK 1.2.0

ECK reached general availability (GA) status with the release of version 1.0.0. The latest available GA version is 1.2.0. It is compatible with the previous GA releases (1.0.x and 1.1.x) and the beta release (1.0.0-beta1), and can be upgraded in-place (with a few exceptions) by applying the new set of deployment manifests. Previous alpha releases, up to and including version 0.9.0, are not compatible with the GA and beta releases and require extra work to upgrade.

Before upgrading, refer to the release notes to make sure that the release does not contain any breaking changes that could affect you. The release highlights document provides more details and possible workarounds for any breaking changes or known issues in each release.

Note that the release notes and highlights only list the changes since the last release. If you are skipping over any intermediate versions during the upgrade — such as going directly from 1.0.0-beta1 to 1.2.0 — review the release notes and highlights of each of the skipped releases to fully understand all the breaking changes you might encounter during and after the upgrade.

## Upgrade from beta or previous GA releases to ECK 1.2.0

The upgrade process results in an update to all the existing managed resources. This triggers a rolling restart of all Elasticsearch and Kibana pods. Guidance is available on controlling this process more gracefully.

If you are running Openshift 3.11 or a Kubernetes version older than 1.13, skip to the upgrade instructions for older Kubernetes releases.

Follow the instructions in Deploy ECK in your Kubernetes cluster to upgrade ECK to the latest version. This will update the ECK installation to the latest binary and update the CRDs and other ECK resources in the cluster. If you are upgrading from the beta version, ensure that your Elasticsearch, Kibana, and APM Server manifests are updated to use the v1 API version instead of v1beta1 after the upgrade.

## Control rolling restarts during the upgrade

Upgrading the operator results in a one-time update to existing managed resources in the cluster. This triggers a rolling restart of pods by Kubernetes to apply those changes. If you have a very large Elasticsearch cluster or multiple Elastic Stack deployments, this rolling restart might be disruptive or inconvenient. To have more control over when the pods belonging to a particular deployment should be restarted, you can add an annotation to the corresponding Elasticsearch, Kibana, or APM Server resources to temporarily exclude them from being managed by the operator. When the time is convenient, you can remove the annotation and let the rolling restart go through.

Once a resource is excluded from being managed by ECK, you will not be able to add/remove nodes, upgrade Stack version, or perform other orchestration tasks by updating the resource manifest. You must remember to remove the exclusion to ensure that your Elastic Stack deployment is continually monitored and managed by the operator.

Exclude Elastic resources from being managed by the operator.

ANNOTATION='common.k8s.elastic.co/pause=true'

# Exclude a single Elasticsearch resource named "quickstart"

kubectl annotate --overwrite elasticsearch quickstart \$ANNOTATION

# Exclude all resources in the current namespace

kubectl annotate --overwrite elastic --all \$ANNOTATION

# Exclude all resources in all of the namespaces:

for NS in $(kubectl get ns -o=custom-columns='NAME:.metadata.name' --no-headers); do kubectl annotate --overwrite elastic --all $ANNOTATION -n \$NS; done

Since ECK 1.1.0, this annotation has been superseded by eck.k8s.elastic.co/managed=false.

Once the operator has been upgraded and you are ready to let the resource become managed again (triggering a rolling restart of pods in the process), remove the annotation.

Resume Elastic resource management by the operator.

RM_ANNOTATION='common.k8s.elastic.co/pause-'

# Resume management of a single Elasticsearch cluster named "quickstart"

kubectl annotate elasticsearch quickstart \$RM_ANNOTATION

Since ECK 1.1.0, this annotation has been superseded by eck.k8s.elastic.co/managed-.

The ECK source repository contains a shell script to assist with mass addition/deletion of annotations.

## Upgrade from beta to GA on older versions of Kubernetes

If you are running Openshift 3.11 or a Kubernetes version older than 1.13 and have existing resources created using the v1beta1 API version, in-place upgrade is not possible due to an upstream bug. You have to completely uninstall the existing ECK operator (which will cause all existing Elastic Stack applications to be deleted) and install the new version of the operator. Make sure to backup your existing manifests and take snapshots of Elasticsearch clusters before uninstalling ECK. Then you can install the new version of ECK, update your manifests to the v1 API version, re-create the Elastic Stack deployments, and restore data from the snapshots.
