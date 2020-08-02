---
title: "网络策略"
linkTitle: ""
weight: 3
description: >
  Webhooks require network connectivity between the Kubernetes API server and the operator.
---

If the creation of an Elasticsearch resource times out with an error message similar to the following, then the Kubernetes API server might be unable to connect to the webhook to validate the manifest.

Error from server (Timeout): error when creating "elasticsearch.yaml": Timeout: request did not complete within requested timeout 30s
If you get this error, try re-running the command with a higher request timeout as follows:

kubectl --request-timeout=1m apply -f elasticsearch.yaml
As the default failurePolicy of the webhook is Ignore, the above command should succeed after about 30 seconds. This is an indication that the API server cannot contact the webhook server and has foregone validation when creating the resource. It is possible that a network policy is blocking any incoming requests to the webhook server. Consult your system administrator to determine whether that is the case, and create an appropriate policy to allow communication between the Kubernetes API server and the webhook server. For example, the following network policy simply opens up the webhook port to the world:

kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
name: allow-webhook-access-from-any
namespace: elastic-system
spec:
podSelector:
matchLabels:
control-plane: elastic-operator
ingress:

- from: []
  ports: - port: 9443
  If you want to restrict the webhook access only to the Kubernetes API server, you must know the IP address of the API server, that you can obtain through this command:

kubectl cluster-info | grep master
Assuming that the API server IP address is 10.1.0.1, the following policy restricts webhook access to just the API server.

kind: NetworkPolicy
apiVersion: networking.k8s.io/v1
metadata:
name: allow-webhook-access-from-apiserver
namespace: elastic-system
spec:
podSelector:
matchLabels:
control-plane: elastic-operator
ingress:

- from:
  - ipBlock:
    cidr: 10.1.0.1/32
    ports:
  - port: 9443
