---
title: "Beats highlights"
linkTitle: ""
weight: 42
---

This list summarizes the most important enhancements in Beats. For the complete list, go to Beats release highlights.

## Support for monitoring Google Cloud service metrics

In this release, we’ve added support for monitoring Google Cloud operations suite (formerly Stackdriver). The new stackdriver metricset in the Google Cloud Platform module collects any service metrics from Google Cloud by using the ListTimeSeries API call.

For a full list of metric types that Google Cloud monitoring supports, see the Google Cloud metrics documentation.

## Specialized Linux integration

To simplify Linux monitoring, we’ve introduced a new Linux module that contains metrics exclusive to the Linux kernel and various subsystems. This is done primarily to avoid cluttering the System module with metricsets that are not broadly cross-compatible. The new Linux module currently includes these metricsets: pageinfo, ksm, and conntrack.
