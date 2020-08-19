---
title: "连接到APM服务器"
linkTitle: ""
weight: 3
---

## APM Server service

The APM Server is exposed with a Service. For information on accessing it, see How to access Elastic Stack services.

To retrieve the list of all the APM Services, use the following command:

kubectl get service --selector='common.k8s.elastic.co/type=apm-server'
NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
apm-server-quickstart-apm-http ClusterIP 10.0.1.252 <none> 8200/TCP 154m

## APM Server secret token

The operator generates an authorization token that agents must send to authenticate themselves to the APM Server.

This token is stored in a secret named {APM-server-name}-apm-token and can be retrieved with the following command:

kubectl get secret/apm-server-quickstart-apm-token -o go-template='{{index .data "secret-token" | base64decode}}'
For more information, see APM Server Reference.
