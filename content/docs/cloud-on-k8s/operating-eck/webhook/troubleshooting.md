---
title: "故障排除"
linkTitle: ""
weight: 2
description: >
  On startup, the operator deploys an admission webhook that points to the operator’s service.
---

If this is inaccessible, you may see errors in your Kubernetes API server logs indicating that it cannot reach the service. A common cause may be that the operator pods are failing to start for some reason, or that the control plane is isolated from the operator pod by some mechanism (for instance via network policies or running the control plane externally as in issue #869 and issue #1369).

For troubleshooting, you can change the failurePolicy of the webhook configuration to Fail, which will cause creations and updates to error out if there is an error contacting the webhook.

On GKE private clusters, requests for creating or updating Elastic resources might take a long time to complete or time out. In this case, you have to add a firewall rule allowing port 9443 from the API server so that it can contact the webhook. See the GKE documentation on adding rules and the Kubernetes issue for more details.

Refer to Network policies for more information about network policies that might be preventing communication between the Kubernetes API server and the webhook server.

## alidation failures

If the validation webhook is preventing you from making changes due to the unknown fields validation like in the example below, you can force the webhook to ignore it by removing the`kubectl.kubernetes.io/last-applied-configuration` annotation from your resource.

```
admission webhook "elastic-es-validation-v1.k8s.elastic.co" denied the request: Elasticsearch.elasticsearch.k8s.elastic.co "quickstart" is invalid: some-misspelled-field: Invalid value: "some-misspelled-field": some-misspelled-field field found in the kubectl.kubernetes.io/last-applied-configuration annotation is unknown
```
