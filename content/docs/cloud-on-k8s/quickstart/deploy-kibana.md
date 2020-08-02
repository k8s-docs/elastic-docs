---
title: "部署Kibana实例"
linkTitle: "部署KB"
weight: 3
---

要部署 Kibana 实例经过以下步骤.

指定 Kibana 实例 并与您的 Elasticsearch 集群关联:

```sh
cat <<EOF | kubectl apply -f -
apiVersion: kibana.k8s.elastic.co/v1
kind: Kibana
metadata:
name: quickstart
spec:
version: 7.8.1
count: 1
elasticsearchRef:
name: quickstart
EOF
```

监控 Kibana 健康和创建进度.

类似 Elasticsearch，您可以检索有关 Kibana 实例的详细信息:

```
kubectl get kibana
```

以及相关的`Pods`:

```sh
kubectl get pod --selector='kibana.k8s.elastic.co/name=quickstart'
```

访问 Kibana.

一个 ClusterIP 服务将自动为 Kibana 创建:

```sh
kubectl get service quickstart-kb-http
```

用 kubectl port-forward 从本地工作站访问 Kibana :

```sh
kubectl port-forward service/quickstart-kb-http 5601
```

在您的浏览器打开 https://localhost:5601 .
您的浏览器会显示一个警告 因为默认配置自签名证书不是由已知证书颁发机构核实并且你的浏览器不信任.
为这个快速启动你可以暂时确认警告但强烈建议您在任何生产部署配置有效证件.

登录为`elastic`用户. 密码可以通过下面的命令来获得:

```
kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo
```
