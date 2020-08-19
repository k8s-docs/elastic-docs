---
title: "使用由ECK管理的Elasticsearch集群"
linkTitle: ""
weight: 3
---

Managing both APM Server and Elasticsearch by ECK allows a smooth and secured integration between the two. The output configuration of the APM Server is setup automatically to establish a trust relationship with Elasticsearch.

To deploy an APM Server and connect it to the cluster quickstart created in the quickstart, apply the following specification:

```sh
cat <<EOF | kubectl apply -f -
apiVersion: apm.k8s.elastic.co/v1
kind: ApmServer
metadata:
name: apm-server-quickstart
namespace: default
spec:
version: 7.8.1
count: 1
elasticsearchRef:
name: quickstart
EOF
```

Monitor APM Server deployment.

You can retrieve details about the APM Server instance:

```sh
kubectl get apmservers
NAME HEALTH NODES VERSION AGE
apm-server-quickstart green 1 7.8.1 8m
```

And you can list all the Pods belonging to a given deployment:

```sh
kubectl get pods --selector='apm.k8s.elastic.co/name=apm-server-quickstart'
NAME READY STATUS RESTARTS AGE
apm-server-quickstart-apm-server-69b447ddc5-fflc6 1/1 Running 0 2m50s
```
