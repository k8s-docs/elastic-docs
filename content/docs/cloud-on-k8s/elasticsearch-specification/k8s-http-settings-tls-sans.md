HTTP settings and TLS SANsedit
Change Kubernetes service typeedit
The default service (<cluster_name>-es-http) created by the operator is a ClusterIP service that makes the Elasticsearch cluster accessible from within the Kubernetes cluster. You can change the service type by updating the spec.http.service.spec.type field as follows:

apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
name: quickstart
spec:
version: 7.8.1
http:
service:
spec:
type: LoadBalancer
nodeSets:

- name: default
  count: 3
  Refer to the Kubernetes service types documentation for more information about different service types available.

When you change the clusterIP setting of the service, ECK will delete and re-create the service as clusterIP is an immutable field. Depending on your client implementation, this might result in a short disruption until the service DNS entries refresh to point to the new endpoints.

Change exposed Elasticsearch nodesedit
By default, the service created by the operator encompasses all Elasticsearch nodes in the cluster. If you prefer to restrict the set of accessible Elasticsearch nodes, you can do so by specifying a new selector. For example, to exclude master nodes from receiving requests, override the service definition as follows:

apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
name: quickstart
spec:
version: 7.8.1
http:
service:
spec:
selector:
elasticsearch.k8s.elastic.co/cluster-name: "quickstart"
elasticsearch.k8s.elastic.co/node-master: "false"
nodeSets:

- name: master
  count: 1
  config:
  node.master: true
  node.data: false
  node.ingest: false
  node.ml: false
- name: data
  count: 5
  config:
  node.master: false
  node.data: true
  node.ingest: false
  node.ml: false
  See Traffic Splitting for information about more advanced traffic-splitting configurations.

Customize the self-signed certificateedit
The operator generates a self-signed TLS certificate to secure the Elasticsearch HTTP layer. You can add extra IP addresses or DNS names to the generated certificate as follows:

apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
name: quickstart
spec:
version: 7.8.1
http:
tls:
selfSignedCertificate:
subjectAltNames: - ip: 1.2.3.4 - dns: hulk.example.com
nodeSets:

- name: default
  count: 3
