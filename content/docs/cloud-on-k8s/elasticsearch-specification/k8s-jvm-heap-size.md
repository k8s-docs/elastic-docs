JVM heap sizeedit
To change the heap size of Elasticsearch, set the ES_JAVA_OPTS environment variable in the podTemplate. It is also highly recommended to set the resource requests and limits at the same time to ensure that the pod gets enough resources allocated within the Kubernetes cluster. See Set compute resources for Elasticsearch for an example and more information.

If ES_JAVA_OPTS is not defined, the Elasticsearch default heap size of 1Gi will be in effect.

See also: Elasticsearch documentation on setting the heap size
