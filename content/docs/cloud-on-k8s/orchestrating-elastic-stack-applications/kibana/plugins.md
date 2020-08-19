---
title: "安装Kibana插件"
linkTitle: ""
weight: 1
---

You can override the Kibana container image to use your own image with the plugins already installed, as described in the custom images doc. You will also want to run an optimize step as part of the build, otherwise it may need to run at startup which requires additional time and resources. Here is a simple example Dockerfile example:

```docker
FROM docker.elastic.co/kibana/kibana:7.8.1
RUN /usr/share/kibana/bin/kibana-plugin install \$PLUGIN_URL
RUN /usr/share/kibana/bin/kibana --optimize
```
