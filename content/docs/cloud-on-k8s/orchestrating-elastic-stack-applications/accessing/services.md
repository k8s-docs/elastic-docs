Servicesedit
You can access Elastic resources by using native Kubernetes services that are not reachable from the public Internet by default.

Manage Kubernetes servicesedit
For each resource, the operator manages a Kubernetes service named <name>-[es|kb|apm|ent]-http, which is of type ClusterIP by default. ClusterIP exposes the service on a cluster-internal IP and makes the service only reachable from the cluster.

> kubectl get svc

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
hulk-apm-http ClusterIP 10.19.212.105 <none> 8200/TCP 1m
hulk-es-http ClusterIP 10.19.252.160 <none> 9200/TCP 1m
hulk-kb-http ClusterIP 10.19.247.151 <none> 5601/TCP 1m
Allow public accessedit
You can expose services in different ways by specifying an http.service.spec.type in the spec of the resource manifest. On cloud providers which support external load balancers, you can set the type field to LoadBalancer to provision a load balancer for the Service, and populate the column EXTERNAL-IP after a short delay. Depending on the cloud provider, it may incur costs.

By default, the Elasticsearch service created by ECK is configured to route traffic to all Elasticsearch nodes in the cluster. Depending on your cluster configuration, you may want more control over the set of nodes that handle different types of traffic (query, ingest, and so on). See Traffic Splitting for more information.

apiVersion: <kind>.k8s.elastic.co/v1
kind: <Kind>
metadata:
name: hulk
spec:
version: 7.8.1
http:
service:
spec:
type: LoadBalancer

> kubectl get svc

NAME TYPE CLUSTER-IP EXTERNAL-IP PORT(S) AGE
hulk-apm-http LoadBalancer 10.19.212.105 35.176.227.106 8200:31000/TCP 1m
hulk-es-http LoadBalancer 10.19.252.160 35.198.131.115 9200:31320/TCP 1m
hulk-kb-http LoadBalancer 10.19.247.151 35.242.197.228 5601:31380/TCP 1m
