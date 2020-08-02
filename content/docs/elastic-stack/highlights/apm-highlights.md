---
title: "APM highlights"
linkTitle: ""
weight: 41
---

This list summarizes the most important enhancements in APM. For the complete list, go to APM release highlights.

## 服务地图

Service maps now integrate with machine learning, offering real-time health indicators based on anomaly detection scores, to help you quickly assess the status and health of your services.

Specifically, machine learning jobs can be created to calculate anomaly scores on APM transaction durations within the selected service. When these jobs are active, service maps will display a color-coded anomaly indicator based on the detected anomaly score. From these scores, you can jump straight to the anomaly detection metric viewier in the Machine learning app to learn more.

APM Service maps
See service maps for more information.

## 部署注释

For enhanced visibility into your deployments, we offer deployment annotations on all transaction charts. Starting in 7.8, you can now explicitly create deployment annotations with our annotation API. The API can integrate into your CI/CD pipeline, so that each time you deploy, a POST request is sent to the annotation API endpoint:

```sh
curl -X POST \
 http://localhost:5601/api/apm/services/${SERVICE_NAME}/annotation \
-H 'Content-Type: application/json' \
-H 'kbn-xsrf: true' \
-H 'Authorization: Basic ${API_KEY}' \
-d '{
      "@timestamp": "${DEPLOY_TIME}",
"service": {
"version": "${SERVICE_VERSION}"
      },
      "message": "${MESSAGE}"
}'
```

See Track deployments with annotations for more information.

## RUM + Elastic Maps

There is now a dedicated “Observability” solution layer in the Maps app. Import your data in just three clicks to begin exploring.

Learn more about Elastic Maps in the documentation.

## RUM 事件负载压缩

APM Server has a new event intake API for RUM events. This intake API supports compressed event payloads from the JavaScript RUM Agent. The difference is impressive—​a reduction of around ~45% in the payload size for an average web page!

Configure the api-version in your RUM Agent to begin using this feature.

## .NET Agent SqlClient support

Auto instrumentation for System.Data.SqlClient is now available for both .NET Core and .NET Framework applications. This means you can get out-of-the-box visibility, including service maps and distributed traces, for the SqlClient calls made from your .NET applications.

Learn more in set up SQLClient, and upgrade to the latest version of the .NET agent to get started.
