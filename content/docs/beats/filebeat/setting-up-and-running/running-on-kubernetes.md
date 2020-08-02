---
title: "在Kubernetes上运行Filebeat"
linkTitle: "K8S上运行"
weight: 1
---

您可以在 Kubernetes 上使用 Filebeat Docker 镜像检索和承载容器日志.

在 Kubernetes 运行弹性云？查看 ECK 运行 Beats。

## Kubernetes 部署 manifests

You deploy Filebeat as a DaemonSet to ensure there’s a running instance on each node of the cluster.

The Docker logs host folder (/var/lib/docker/containers) is mounted on the Filebeat container. Filebeat starts an input for the files and begins harvesting them as soon as they appear in the folder.

Everything is deployed under the kube-system namespace by default. To change the namespace, modify the manifest file.

To download the manifest file, run:

```sh
curl -L -O https://raw.githubusercontent.com/elastic/beats/7.8/deploy/kubernetes/filebeat-kubernetes.yaml
```

If you are using Kubernetes 1.7 or earlier: Filebeat uses a hostPath volume to persist internal data. It’s located under /var/lib/filebeat-data. The manifest uses folder autocreation (DirectoryOrCreate), which was introduced in Kubernetes 1.8. You need to remove type: DirectoryOrCreate from the manifest and create the host folder yourself.

## 设置

By default, Filebeat sends events to an existing Elasticsearch deployment, if present. To specify a different destination, change the following parameters in the manifest file:

- name: ELASTICSEARCH_HOST
  value: elasticsearch
- name: ELASTICSEARCH_PORT
  value: "9200"
- name: ELASTICSEARCH_USERNAME
  value: elastic
- name: ELASTICSEARCH_PASSWORD
  value: changeme
  Red Hat OpenShift configuration
  If you are using Red Hat OpenShift, you need to specify additional settings in the manifest file and enable the container to run as privileged.

Modify the DaemonSet container spec in the manifest file:

securityContext:
runAsUser: 0
privileged: true
Grant the filebeat service account access to the privileged SCC:

oc adm policy add-scc-to-user privileged system:serviceaccount:kube-system:filebeat
This command enables the container to be privileged as an administrator for OpenShift.

Override the default node selector for the kube-system namespace (or your custom namespace) to allow for scheduling on any node:

oc patch namespace kube-system -p \
'{"metadata": {"annotations": {"openshift.io/node-selector": ""}}}'
This command sets the node selector for the project to an empty string. If you don’t run this command, the default node selector will skip master nodes.

## 部署

To deploy Filebeat to Kubernetes, run:

kubectl create -f filebeat-kubernetes.yaml
To check the status, run:

```sh
kubectl --namespace=kube-system get ds/filebeat
```

NAME DESIRED CURRENT READY UP-TO-DATE AVAILABLE NODE-SELECTOR AGE
filebeat 32 32 0 32 0 <none> 1m
Log events should start flowing to Elasticsearch. The events are annotated with metadata added by the add_kubernetes_metadata processor.
