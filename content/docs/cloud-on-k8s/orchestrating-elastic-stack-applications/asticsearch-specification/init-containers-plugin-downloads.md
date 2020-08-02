---
title: "为插件下载初始化容器"
linkTitle: "插件下载"
weight: 10
---

You can install custom plugins before the Elasticsearch container starts with an initContainer. For example:

```yaml
spec:
  nodeSets:
    - name: default
      count: 3
      podTemplate:
        spec:
          initContainers:
            - name: install-plugins
              command:
                - sh
                - -c
                - |
                  bin/elasticsearch-plugin install --batch repository-gcs
```

You can also override the Elasticsearch container image to use your own image with the plugins already installed, as described in the custom images doc. The snapshots document has more information on both these options. The Kubernetes document on init containers has more information on their usage as well.

The init container inherits the image of the main container image if one is not explicitly set. It also inherits the volume mounts as long as the name and mount path do not conflict. It also inherits the Pod name and IP address environment variables.
