---
title: "快速开始"
linkTitle: ""
weight: 1
description: 应用下面的规格来部署 Filebeat 和收集运行在集群 Kubernetes 的所有容器的日志.
---

ECK 自动配置名为 quickstart 的 Elasticsearch 集群的安全连接, 创建在[Elasticsearch 快速入门](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-quickstart.html).

```sh
cat <<EOF | kubectl apply -f -
apiVersion: beat.k8s.elastic.co/v1beta1
kind: Beat
metadata:
  name: quickstart
spec:
  type: filebeat
  version: 7.8.1
  elasticsearchRef:
    name: quickstart
  config:
    filebeat.inputs:
    - type: container
      paths:
      - /var/log/containers/*.log
  daemonSet:
    podTemplate:
      spec:
        dnsPolicy: ClusterFirstWithHostNet
        hostNetwork: true
        securityContext:
          runAsUser: 0
        containers:
        - name: filebeat
          volumeMounts:
          - name: varlogcontainers
            mountPath: /var/log/containers
          - name: varlogpods
            mountPath: /var/log/pods
          - name: varlibdockercontainers
            mountPath: /var/lib/docker/containers
        volumes:
        - name: varlogcontainers
          hostPath:
            path: /var/log/containers
        - name: varlogpods
          hostPath:
            path: /var/log/pods
        - name: varlibdockercontainers
          hostPath:
            path: /var/lib/docker/containers
EOF
```

参见[配置示例](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-beat-configuration-examples.html)对于更愿意使用的`manifests`.

1. 监控 Beats

   检索有关 Filebeat 细节:

   ```sh
   kubectl get beat
   ```

   ```sh
   NAME                  HEALTH   AVAILABLE   EXPECTED   TYPE       VERSION   AGE
   quickstart            green    3           3          filebeat   7.8.1     2m
   ```

   列出所有`Pods`属于给定`Beat`:

   ```sh
   kubectl get pods --selector='beat.k8s.elastic.co/name=quickstart-beat-filebeat'
   ```

   ```sh
   NAME                                      READY   STATUS    RESTARTS   AGE
   quickstart-beat-filebeat-tkz65            1/1     Running   0          3m45s
   quickstart-beat-filebeat-kx5jt            1/1     Running   0          3m45s
   quickstart-beat-filebeat-nb6qh            1/1     Running   0          3m45s
   ```

2. 访问 Pods 之一的日志

   ```sh
   kubectl logs -f quickstart-beat-filebeat-tkz65
   ```

3. 访问由 Filebeat 摄入日志

   - 依据 Elasticsearch 部署[指南](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-elasticsearch.html)并运行:

     ```
     curl -u "elastic:$PASSWORD" -k "https://localhost:9200/filebeat-*/_search"
     ```

   - 或者按照 Kibana 部署[指南](https://www.elastic.co/guide/en/cloud-on-k8s/current/k8s-deploy-kibana.html), 登录并进入 Kibana > Discover.
