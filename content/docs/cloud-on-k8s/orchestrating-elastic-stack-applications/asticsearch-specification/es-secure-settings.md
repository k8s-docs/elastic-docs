---
title: "安全设置"
linkTitle: ""
weight: 8
---

You can specify secure settings with Kubernetes secrets. The secrets should contain a key-value pair for each secure setting you want to add. ECK automatically injects these settings into the keystore on each Elasticsearch node before it starts Elasticsearch.

```yaml
spec:
  secureSettings:
    - secretName: one-secure-settings-secret
    - secretName: two-secure-settings-secret
```

You can export a subset of secret keys and also project keys to specific paths using the entries, key and path fields:

```yaml
spec:
  secureSettings:
    - secretName: your-secure-settings-secret
      entries:
        - key: key1
        - key: key2
          path: newkey2
```

See How to create automated snapshots for an example use case.
