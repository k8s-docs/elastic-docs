---
title: "在ECK上运行APM服务器"
linkTitle: "运行APM"
weight: 3
---

This section describes how to deploy, configure and access an APM Server with ECK.

Use an Elasticsearch cluster managed by ECK
Advanced configuration

Customize the APM Server configuration
APM Secrets keystore for secure settings
Reference an existing Elasticsearch cluster
TLS Certificates
Connect to the APM Server

APM Server service
APM Server secret token
The current Docker image of the APM Server must run as root or with the user id 1000. This prevents the APM Server from running in some environments such as OpenShift, or on any Kubernetes cluster that would set a different user in the security context.
