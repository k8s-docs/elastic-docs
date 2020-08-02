Virtual memoryedit
By default, Elasticsearch uses memory mapping (mmap) to efficiently access indices. Usually, default values for virtual address space on Linux distributions are too low for Elasticsearch to work properly, which may result in out-of-memory exceptions. This is why the quickstart example disables mmap via the node.store.allow_mmap: false setting. For production workloads, it is strongly recommended to increase the kernel setting vm.max_map_count to 262144 and leave node.store.allow_mmap unset.

The kernel setting vm.max_map_count=262144 can be set on the host either directly or by a dedicated init container, which must be privileged. To add an init container that changes the host kernel setting before your Elasticsearch pod starts, you can use the following example Elasticsearch spec:

cat <<EOF | kubectl apply -f -
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
name: quickstart
spec:
version: 7.8.1
nodeSets:

- name: default
  count: 3
  config:
  node.master: true
  node.data: true
  node.ingest: true
  podTemplate:
  spec:
  initContainers: - name: sysctl
  securityContext:
  privileged: true
  command: ['sh', '-c', 'sysctl -w vm.max_map_count=262144']
  EOF
  Note that this requires the ability to run privileged containers, which is likely not the case on many secure clusters.

For more information, see the Elasticsearch documentation on Virtual memory.

Optionally, you can select a different type of file system implementation for the storage. For possible options, see the store module documentation.

spec:
nodeSets:

- name: default
  count: 3
  config:
  index.store.type: niofs
