Transport settingsedit
The transport module in Elasticsearch is used for internal communication between nodes within the cluster as well as communication between remote clusters. See the Elasticsearch documentation for details.

In the spec.transport.service. section, you can change the Kubernetes service used to expose the Elasticsearch transport module:

spec:
transport:
service:
metadata:
labels:
my-custom: label
spec:
type: LoadBalancer
Check the Kubernetes Publishing Services (ServiceTypes) that are currently available.

Please note that when you change the clusterIP setting of the service, ECK will delete and re-create the service as clusterIP is an immutable field. This does not typically have an impact on connectivity as the transport module uses long-lived TCP connections, but may cause a small network disruption between Elasticsearch nodes.
