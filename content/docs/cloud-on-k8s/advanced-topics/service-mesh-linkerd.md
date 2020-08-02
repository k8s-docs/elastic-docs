Linkerdedit
The following sections describe how to connect the operator and managed resources to the Linkerd service mesh. It is assumed that Linkerd is already installed and configured on your Kubernetes cluster. If you are new to Linkerd, refer to the product documentation for more information and installation instructions.

These instructions have been tested with Linkerd 2.7.0.

Connect the operator to the Linkerd service meshedit
In order to connect the operator to the service mesh, Linkerd sidecar must be injected into the ECK deployment. This can be done during installation as follows:

linkerd inject https://download.elastic.co/downloads/eck/1.2.0/all-in-one.yaml | kubectl apply -f -
Confirm that the operator is now meshed:

linkerd stat sts/elastic-operator -n elastic-system
If the installation was successful, the output of the above command should show 1/1 under the MESHED column.

Connect Elastic Stack applications to the Linkerd service meshedit
The easiest way to connect applications to the service mesh is by adding the linkerd.io/inject: enabled annotation to the deployment namespace. For example, if you are planning to deploy Elastic Stack applications to a namespace named elastic-stack, annotate it as follows to enable automatic Linkerd sidecar injection.

kubectl annotate namespace elastic-stack linkerd.io/inject=enabled
Any Elasticsearch, Kibana, or APM Server resources deployed to a namespace with the above annotation will automatically join the mesh.

Alternatively, if you only want specific resources to join the mesh, add the linkerd.io/inject: enabled annotation to the podTemplate (see API documentation) of the resource as follows:

podTemplate:
metadata:
annotations:
linkerd.io/inject: enabled
If automatic sidecar injection is enabled and auto mounting of service account tokens is not disabled on your Kubernetes cluster, examples defined elsewhere in the ECK documentation will continue to work under Linkerd without requiring any modifications. However, as the default behaviour of ECK is to enable TLS for Elasticsearch, Kibana and APM Server resources, you will not be able to view detailed traffic information from Linkerd dashboards and command-line utilities. The following sections illustrate the optional configuration necessary to enhance the integration of Elastic Stack applications with Linkerd.

Elasticsearchedit
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
name: elastic-linkerd
spec:
version: 7.8.1
http:
tls:
selfSignedCertificate:
disabled: true
nodeSets:

- name: default
  count: 3
  config:
  node.store.allow_mmap: false
  podTemplate:
  metadata:
  annotations:
  linkerd.io/inject: enabled
  spec:
  automountServiceAccountToken: true

Disable automatic TLS to allow Linkerd to gather more statistics about connections (optional).

Explicitly enable sidecar injection (optional if the namespace is already annotated).

Enable service account token mounting to provide service identity (only required to enable mTLS if service account auto-mounting is disabled).

Kibana and APM Serveredit
The configuration is almost identical for Kibana and APM Server resources.

apiVersion: ...
kind: ...
metadata:
name: elastic-linkerd
spec:
version: 7.8.1
count: 1
elasticsearchRef:
name: elastic-linkerd
http:
tls:
selfSignedCertificate:
disabled: true
podTemplate:
metadata:
annotations:
linkerd.io/inject: enabled
spec:
automountServiceAccountToken: true

Disable automatic TLS to allow Linkerd to gather more statistics about connections (optional).

Explicitly enable sidecar injection (optional if the namespace is already annotated).

Enable service account token mounting to provide service identity (only required to enable mTLS if service account auto-mounting is disabled).
