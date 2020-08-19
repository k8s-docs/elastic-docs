---
title: "升级弹性堆栈版本"
linkTitle: ""
weight: 2
---

The operator can safely perform upgrades to newer versions of the various Elastic Stack resources.

Follow the instructions in the Elasticsearch documentation. Make sure that your cluster is compatible with the target version, take backups, and follow the specific upgrade instructions for each resource type, especially the order in which the upgrade should be carried out. When you are ready, modify the version field in the resource spec to the desired stack version and the operator will start the upgrade process automatically.

See Nodes orchestration for more information on how the operator performs upgrades and how to tune its behavior.
