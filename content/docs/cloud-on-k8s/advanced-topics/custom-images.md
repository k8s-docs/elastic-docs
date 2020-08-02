Create custom imagesedit
You can create your own custom Elasticsearch or Kibana image instead of using the base image provided by Elastic. You might want to do this to preload plugins in the image rather than having to install them via init container each time a pod starts. To do this, you must use the official image as the base for it to function properly. For example, if you want to create an Elasticsearch 7.8.1 image with the Google Cloud Storage Repository Plugin, you can do the following:

Create a Dockerfile containing:

FROM docker.elastic.co/elasticsearch/elasticsearch:7.8.1
RUN bin/elasticsearch-plugin install --batch repository-gcs
Build the image with:

docker build --tag elasticsearch-gcs:7.8.1
There are various hosting options for your images. If you use Google Kubernetes Engine, it is automatically configured to use the Google Container Registry (see here for more information). To use the image, you can then push to the registry with:

docker tag elasticsearch-gcs:7.8.1 gcr.io/$PROJECT-ID/elasticsearch-gcs:7.8.1
docker push gcr.io/$PROJECT-ID/elasticsearch-gcs:7.8.1
Configure your Elasticsearch specification to use the newly pushed image, for example:

spec:
version: 7.8.1
image: gcr.io/\$PROJECT-ID/elasticsearch-gcs:7.8.1
Providing the correct version is always required as ECK reasons about APIs and capabilities available to it based on the version field.

The steps are similar for Azure Kubernetes Service and AWS Elastic Container Registry.

For more information, you can check the following references:

Elasticsearch doc on creating custom images
Google Container Registry docs
Azure Container Registry docs
Amazon Elastic Container Registry docs
OpenShift Container Platform registry docs
Override the default container registryedit
When creating Elasticsearch, Kibana, or APM Server resources, the operator defaults to using container images pulled from the docker.elastic.co registry. If you are in an environment where external network access is restricted, you could configure the operator to use a different default container registry by starting the operator with the --container-registry command-line flag. (See Configure ECK for more information about how to configure the operator using command-line flags and environment variables).

The operator expects container images to be located at specific paths in the default container registry. Make sure that your container images are stored at the right path and are tagged correctly with the stack version number. For example, if your private registry is my.registry and you wish to deploy components from stack version 7.8.1, the following image paths should exist:

my.registry/elasticsearch/elasticsearch:7.8.1
my.registry/kibana/kibana:7.8.1
my.registry/apm/apm-server:7.8.1
