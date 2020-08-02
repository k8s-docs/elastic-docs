---
title: "Pod转钩"
linkTitle: ""
weight: 18
---

When an Elasticsearch Pod is terminated, its Endpoint is removed from the Service and the Elasticsearch process is terminated. As these two operations happen in parallel, a race condition exists. If the Elasticsearch process is already shut down, but the Endpoint is still a part of the Service, any new connection might fail. For more information, see Termination of pods.

Moreover, kube-proxy resynchronizes its rules every 30 seconds by default. During that 30 second time window, the terminating Pod IP may still be used when targeting the service. Please note the resync operation itself may take some time, especially if kube-proxy is configured to use iptables with a lot of services and rules to apply.

To address this issue and minimise unavailability, ECK relies on a PreStop lifecycle hook. This waits to terminate the Elasticsearch process until the Service DNS record does not contain the IP of the Pod. First, the PreStop lifecycle hook will keep querying DNS for PRE_STOP_MAX_WAIT_SECONDS (defaulting to 20) until the Pod IP is not referenced anymore. Then, it waits for an additional PRE_STOP_ADDITIONAL_WAIT_SECONDS (defaulting to 30). Additional wait is used to:

give time to in-flight requests to be completed
give clients time to use the terminating Pod IP resolved just before DNS record was updated
give kube-proxy time to refresh ipvs or iptables rules on all nodes, depending on its sync period setting
The exact behavior is configurable using environment variables, for example:

```yaml
spec:
  version: 7.8.1
  nodeSets:
    - name: default
      count: 1
      podTemplate:
        spec:
          containers:
            - name: elasticsearch
              env:
                - name: PRE_STOP_MAX_WAIT_SECONDS
                  value: "10"
                - name: PRE_STOP_ADDITIONAL_WAIT_SECONDS
                  value: "5"
```
