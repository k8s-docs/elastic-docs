Volume claim templatesedit
By default, the operator creates a PersistentVolumeClaim with a capacity of 1Gi for each pod in an Elasticsearch cluster to prevent data loss in case of accidental pod deletion. For production workloads, you should define your own volume claim template with the desired storage capacity and (optionally) the Kubernetes storage class to associate with the persistent volume. The name of the volume claim must always be elasticsearch-data.

spec:
nodeSets:

- name: default
  count: 3
  volumeClaimTemplates: - metadata:
  name: elasticsearch-data
  spec:
  accessModes: - ReadWriteOnce
  resources:
  requests:
  storage: 5Gi
  storageClassName: standard
  ECK automatically deletes PersistentVolumeClaim resources if they are not required for any Elasticsearch node. The corresponding PersistentVolume may be preserved, depending on the configured storage class reclaim policy.

Depending on the Kubernetes configuration and the underlying file system, some persistent volumes cannot be resized after they are created. When you define volume claims, consider future storage requirements and make sure you have enough space to support the expected growth.

If you are not concerned about data loss, you can use an emptyDir volume for Elasticsearch data as well:

spec:
nodeSets:

- name: data
  count: 10
  podTemplate:
  spec:
  volumes: - name: elasticsearch-data
  emptyDir: {}
  Using emptyDir is not recommended due to the high likelihood of permanent data loss.
