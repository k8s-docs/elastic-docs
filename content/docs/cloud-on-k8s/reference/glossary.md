Glossaryedit
This glossary supplements the Kubernetes glossary and covers terms used in the Elastic Cloud on Kubernetes (ECK) documentation.

CA
Certificate Authority. An entity that issues digital certificates to verify identities over a network.
Cluster
Can refer to either an Elasticsearch cluster or a Kubernetes cluster depending on the context.
CRD
Custom Resource Definition. ECK extends the Kubernetes API with CRDs to allow users to deploy and manage Elasticsearch, Kibana and APM server resources just as they would do with built-in Kubernetes resources.
ECK
Elastic Cloud on Kubernetes. Kubernetes operator to orchestrate Elasticsearch, Kibana and APM Server deployments on Kubernetes.
EKS
Elastic Kubernetes Service. Managed Kubernetes service provided by Amazon Web Services (AWS).
GCS
Google Cloud Storage. Block storage service provided by Google Cloud Platform (GCP).
GKE
Google Kubernetes Engine. Managed Kubernetes service provided by Google Cloud Platform (GCP).
K8s
Shortened form (numeronym) of "Kubernetes" derived from replacing "ubernete" with "8".
Node
Can refer to either an Elasticsearch Node or a Kubernetes Node depending on the context. ECK maps an Elasticsearch node to a Kubernetes Pod which can get scheduled onto any available Kubernetes node that can satisfy the resource requirements and node constraints defined in the pod template.
NodeSet
A set of Elasticsearch nodes that share the same Elasticsearch configuration and a Kubernetes Pod template. Multiple NodeSets can be defined in the Elasticsearch CRD to achieve a cluster topology consisting of groups of Elasticsearch nodes with different node roles, resource requirements and hardware configurations (Kubernetes node constraints).
OpenShift
A Kubernetes platform by RedHat.
Operator
A design pattern in Kubernetes for managing custom resources. ECK implements the operator pattern to manage Elasticsearch, Kibana and APM Server resources on Kubernetes.
PDB
Pod Disruption Budget.
PVC
Persistent Volume Claim.
QoS
Quality of Service. When a Kubernetes cluster is under heavy load, the Kubernetes scheduler makes pod eviction decisions based on the QoS class of individual pods. Manage compute resources explains how to define QoS classes for Elasticsearch, Kibana and APM Server pods.
RBAC
Role-based Access Control. A security mechanism in Kubernetes where access to cluster resources is restricted to principals having the appropriate role. See https://kubernetes.io/docs/reference/access-authn-authz/rbac/ for more information.
