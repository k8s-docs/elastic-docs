---
title: "升级弹性堆栈"
linkTitle: ""
weight: 3
---

When upgrading to a new version of Elasticsearch, you need to upgrade each of the products in your Elastic Stack. Beats and Logstash 6.7 are compatible with Elasticsearch 7.8.1 to give you flexibility in scheduling the upgrade.

Elasticsearch supports rolling upgrades between minor versions, from Elasticsearch 5.6 to 6.8, and from 6.8 to 7.8.1.

5.x indices are not compatible with 7.8.1. You must remove or reindex them to upgrade to 7.8.1. The default Beats and Logstash mapping templates also need to be updated to work with 7.8.1.

## Preparing to upgrade

Before upgrading the Elastic Stack to 7.8.1:

Check the Elasticsearch deprecation log to see if you’re using any deprecated features and update your code accordingly. By default, deprecation warnings are logged when the log level is set to WARN.
Review the Breaking changes and upgrade your code to work with 7.8.1.
Upgrade to 6.8 and use the Kibana Upgrade Assistant to reindex any indices that are not compatible with 7.8.1.

Upgrade Assistant

The Upgrade Assistant and migration APIs are enabled with both the Basic and Trial licenses. You can install the default distribution of 6.8 to use the Upgrade Assistant to prepare to upgrade even if you are upgrading to the OSS distribution of 7.8.1.

Use the Upgrade Assistant to identify any changes you need to make to your cluster configuration.

## Upgrade process

When you’ve made the necessary changes and are ready to upgrade from 6.8 to 7.8.1:

Test the upgrade in a dev environment before upgrading your production cluster.
Back up your data. You cannot roll back to an earlier version unless you have a snapshot of your data. For information about creating snapshots, see Snapshot and restore.
Consider closing machine learning jobs before you start the upgrade process. While machine learning jobs can continue to run during a rolling upgrade, it increases the overhead on the cluster during the upgrade process. For more information, see Rolling upgrades.
Upgrade the components of your Elastic Stack in the following order:

Elasticsearch Hadoop: install instructions
Elasticsearch: upgrade instructions
Kibana: upgrade instructions
Logstash: upgrade instructions
Beats: upgrade instructions
APM Server: upgrade instructions
Logstash 6.8 and Beats 6.8 are compatible with all 7.x versions of Elasticsearch. This provides flexibility in when you schedule the upgrades for your Logstash instances and Beats agents, but we recommend upgrading as soon as possible to take advantage of performance improvements and other enhancements.

## Upgrading from 6.6 or earlier

To upgrade directly to Elasticsearch 7.8.1 from versions 6.0-6.6, you must manually reindex any 5.x indices you need to carry forward, and perform a full cluster restart. This includes any internal indices created in 5.x, such as the .kibana and .security\* indices.

Make sure all 5.x indices have been deleted before upgrading to 7.8.1. Elasticsearch 7.8.1 will fail to start if any 5.x indices are present.

If you are running a version prior to 6.0, upgrade to 6.8 and reindex your old indices or bring up a new 7.8.1 cluster and reindex from remote.

The recommended path is to upgrade to 6.8 before upgrading to 7.8.1. This makes it easier to identify the changes you need to make to upgrade and enables you to perform a rolling upgrade with no downtime.

## Upgrading on Elastic Cloud

A single click in the Elastic Cloud console can upgrade a cluster to a newer version, add more processing capacity, change plugins, and enable or disable high availability, all at the same time. During the upgrade process, Elasticsearch, Kibana, X-Pack and the officially included plugins are upgraded simultaneously.

Although upgrading your Elastic Cloud clusters is easy, you still need to address breaking changes that affect your application. Minor version upgrades, upgrades from 6.8 to 7.8.1, and all other cluster configuration changes can be performed with no downtime.

To avoid downtime when a full cluster restart is required:

Provision an additional cluster with the new Elasticsearch version, reindex your data, and send index requests to both clusters temporarily.
Verify that the new cluster performs as expected, fix any problems, and then permanently swap in the new cluster.
Delete the old cluster to stop incurring additional costs. You are billed only for the time that the new cluster runs in parallel with your old cluster. Usage is billed on an hourly basis.
To learn more about the upgrade process on Elastic Cloud, see Upgrade versions.

Elastic Cloud only supports upgrades to released versions. Preview releases and master snapshots are not supported.
