---
title: "自定义配置文件和插件"
linkTitle: "配置和插件"
weight: 9
---

To run Elasticsearch with specific plugins or configuration files installed on ECK you have two options:

Create a custom Docker image with the plugins or files pre-installed.
Install the plugins or configuration files at Pod startup time.
The first option has the advantage that you can verify the correctness of the image before rolling it out to your ECK installation, while the second option gives you maximum flexibility. But the second option also means you might catch any errors only at runtime. Plugin installation at runtime has another drawback in that it needs access to the Internet from your cluster and downloads each plugin multiple times, once for each Elasticsearch node.

See Creating custom images for instructions on how to build custom Docker images based on the official Elastic images.

The following example describes option 2, using a repository plugin. To install the plugin before the Elasticsearch nodes start, use an init container to run the plugin installation tool.

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
                  bin/elasticsearch-plugin install --batch repository-azure
```

To install custom configuration files you can use volumes and volume mounts.

The next example shows how to add a synonyms file for the synonym token filter in Elasticsearch. But you can use the same approach for any kind of file you want to mount into the configuration directory of Elasticsearch.

```yaml
spec:
  nodeSets:
    - name: default
      count: 3
      podTemplate:
        spec:
          containers:
            - name: elasticsearch
              volumeMounts:
                - name: synonyms
                  mountPath: /usr/share/elasticsearch/config/dictionaries
          volumes:
            - name: synonyms
              configMap:
                name: synonyms
```

Elasticsearch runs by convention in a container called elasticsearch.

Assuming you have created a config map in the same namespace as Elasticsearch with the name synonyms containing the synonyms file(s).
