---
title: "故障排除"
linkTitle: ""
weight: 4
---

## 当指定 kibanaRef 时 Beat Pods 崩溃

When kibanaRef is specified, Beat tries to connect to the Kibana instance. If it’s unable to do so, the Beat process exits and the Pod restarts. This may happen when Kibana is not yet up or when Beat user is not yet created in Elasticsearch. The Pod may restart a few times when it is first deployed. Afterwards, the Beat should run successfully.

## 配置包含关键: null 格式有误

When kubectl is used to modify a resource, it calculates a diff between the user applied and the already existing config. This diff has a special semantics defined, that forces the removal of a particular key by setting its value to null. When user applied config contains some_key: null (or equivalent some_key: ~) it’s interpreted as an instruction to remove some_key, often resulting in a malformed config. In Beat configs, this is often the case with processors. To avoid this:

change some_key: null to some_key: {} - if it doesn’t change config semantics
instead of config use configRef and reference a Secret containing the config - Secret data doesn’t go through the same diffing process
