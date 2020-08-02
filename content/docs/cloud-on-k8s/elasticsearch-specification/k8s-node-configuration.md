Node configurationedit
Any setting defined in the elasticsearch.yml configuration file can also be defined for set of Elasticsearch nodes in the spec.nodeSets[?].config section.

spec:
nodeSets:

- name: masters
  count: 3
  config:
  node.master: true
  node.data: false
  node.ingest: false
  node.ml: false
  xpack.ml.enabled: true
  node.remote_cluster_client: false
- name: data
  count: 10
  config:
  node.master: false
  node.data: true
  node.ingest: true
  node.ml: true
  node.remote_cluster_client: false
  For more information on Elasticsearch settings, see Configuring Elasticsearch.
