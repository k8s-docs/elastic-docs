Traffic Splittingedit
The default Kubernetes service created by ECK — named <cluster_name>-es-http — is configured to include all the Elasticsearch nodes in that cluster. This configuration is good enough to get started and adequate for most simple use cases. However, if you are operating an Elasticsearch cluster with different node types and want control over which nodes handle which types of traffic, you should create additional Kubernetes services yourself to do so. Alternatively, you could make use of features provided by third-party software such as service meshes and ingress controllers to achieve more advanced traffic management configurations.

The recipes directory in the ECK source repository contains several examples of using third-party service meshes and ingress controllers for managing traffic to ECK resources.

The service configurations shown in the following sections are based on this definition of an Elasticsearch cluster:

apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
name: hulk
spec:
version: 7.8.1
nodeSets:

# Dedicated master nodes

- name: master
  count: 3
  config:
  node.master: true
  node.data: false
  node.ingest: false
  node.ml: false

# Dedicated data nodes

- name: data
  count: 6
  config:
  node.master: false
  node.data: true
  node.ingest: false
  node.ml: false

# Dedicated ingest nodes

- name: ingest
  count: 3
  config:
  node.master: false
  node.data: false
  node.ingest: true
  node.ml: false

# Dedicated coordinator nodes

- name: coordinator
  count: 3
  config:
  node.master: false
  node.data: false
  node.ingest: false
  node.ml: false

# Dedicated machine learning nodes

- name: ml
  count: 3
  config:
  node.master: false
  node.data: false
  node.ingest: false
  node.ml: true
  Create services for exposing different node typesedit
  The following examples illustrate how to create services for accessing different types of Elasticsearch nodes. The procedure for exposing services publicly is the same as described in Allow public access.

Coordinator nodes

apiVersion: v1
kind: Service
metadata:
name: hulk-es-coordinator-nodes
spec:
ports: - name: https
port: 9200
targetPort: 9200
selector:
elasticsearch.k8s.elastic.co/cluster-name: "hulk"
elasticsearch.k8s.elastic.co/node-master: "false"
elasticsearch.k8s.elastic.co/node-data: "false"
elasticsearch.k8s.elastic.co/node-ingest: "false"
elasticsearch.k8s.elastic.co/node-ml: "false"
Ingest nodes

apiVersion: v1
kind: Service
metadata:
name: hulk-es-ingest-nodes
spec:
ports: - name: https
port: 9200
targetPort: 9200
selector:
elasticsearch.k8s.elastic.co/cluster-name: "hulk"
elasticsearch.k8s.elastic.co/node-ingest: "true"
Non-master nodes

apiVersion: v1
kind: Service
metadata:
name: hulk-es-non-master-nodes
spec:
ports: - name: https
port: 9200
targetPort: 9200
selector:
elasticsearch.k8s.elastic.co/cluster-name: "hulk"
elasticsearch.k8s.elastic.co/node-master: "false"
