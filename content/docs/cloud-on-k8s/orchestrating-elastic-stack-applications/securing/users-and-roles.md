---
title: "用户和角色"
linkTitle: ""
weight: 2
description: >
  创建自定义用户和角色
---

## 默认 elastic 用户

当创建 Elasticsearch 资源, 一个名为`elastic`默认的用户被自动创建, 并分配超级用户角色.

它的密码可以在 Kubernetes`secret`检索, 他的名字是根据 Elasticsearch 资源名称: <elasticsearch-name>-es-elastic-user.

例如, 对于 Elasticsearch 集群`elastic`用户名为`quickstart`的密码可以被检索:

```sh
kubectl get secret quickstart-es-elastic-user -o go-template='{{.data.elastic | base64decode}}'
```

## 创建自定义用户

### Native realm

您可以使用 Elasticsearch 用户管理 API 在 Elasticsearch Native realm 创建自定义的用户.

### File realm

自定义的用户也可以通过提供所需的 在 Kubernetes secrets 里 的 file realm 内容创建 , 在 Elasticsearch 资源引用.

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch-sample
spec:
  version: 7.8.1
  auth:
    fileRealm:
      - secretName: my-filerealm-secret-1
      - secretName: my-filerealm-secret-2
  nodeSets:
    - name: default
      count: 1
```

您可以在 Elasticsearch 说明书中提及的几个秘密。
ECK 聚合他们的内容合并到一个秘密, 安装在每一个 Elasticsearch Pod.

引用的秘密可以由 2 个的条目.
您可以提供在每个秘密进入任一或两个入口:

- users: 用户文件的内容. 它指定的用户名和密码哈希, 作为文件境界文档中描述.
- users_roles: 该 users_roles 文件的内容. 它相关联的每个角色的用户的列表, 作为文件境界文档中描述.

如果指定了多个用户 在超过一个秘密的同名, 最后一个优先.
如果指定了多个角色 在超过一个秘密的同名, 每个角色的单个条目从从所有的秘密与其对应的用户的级联衍生 .

下面的秘密指定了三个用户和他们各自的角色:

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: my-filerealm-secret
stringData:
  users: |-
    rdeniro:$2a$10$BBJ/ILiyJ1eBTYoRKxkqbuDEdYECplvxnqQ47uiowE7yGqvCEgj9W
    alpacino:$2a$10$cNwHnElYiMYZ/T3K4PvzGeJ1KbpXZp2PfoQD.gfaVdImnHOwIuBKS
    jacknich:{PBKDF2}50000$z1CLJt0MEFjkIK5iEfgvfnA6xq7lF25uasspsTKSo5Q=$XxCVLbaKDimOdyWgLCLJiyoiWpA/XDMe/xtVgn1r5Sg=
  users_roles: |-
    admin:rdeniro
    power_user:alpacino,jacknich
    user:jacknich
```

您可以填写两个用户的内容和 users_roles 使用 elasticsearch-users 工具.

例如，在一个 Docker 容器调用工具:

```sh
# create a folder with the 2 files
mkdir filerealm
touch filerealm/users filerealm/users_roles

# create user 'myuser' with role 'monitoring_user'
docker run \
    -v $(pwd)/filerealm:/usr/share/elasticsearch/config \
    docker.elastic.co/elasticsearch/elasticsearch:7.8.1 \
    bin/elasticsearch-users useradd myuser -p mypassword -r monitoring_user

# create a Kubernetes secret with the file realm content
kubectl create secret generic my-file-realm-secret --from-file filerealm
```

## 创建自定义角色

角色可以使用角色管理 API 指定,或 Kibana 角色管理 UI.

另外, 基于文件的角色管理可以通过参考包含角色规范 Kubernetes 秘密实现 .

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch-sample
spec:
  version: 7.8.1
  auth:
    roles:
      - secretName: my-roles-secret-1
      - secretName: my-roles-secret-2
  nodeSets:
    - name: default
      count: 1
```

几个秘密可以在 Elasticsearch 说明书中引用.
ECK 聚合他们的内容合并到一个秘密，安装在每个 Elasticsearch`Pod`.

每个秘密必须有一个 roles.yml 条目, 包含角色定义.

如果您在不止一个秘密指定名称相同的多个角色, 最后一个优先.

下面的秘密指定一个角色名为`click_admins`:

```yaml
kind: Secret
apiVersion: v1
metadata:
  name: my-roles-secret
stringData:
  roles.yml: |-
    click_admins:
      run_as: [ 'clicks_watcher_1' ]
      cluster: [ 'monitor' ]
      indices:
      - names: [ 'events-*' ]
        privileges: [ 'read' ]
        field_security:
          grant: ['category', '@timestamp', 'message' ]
        query: '{"match": {"category": "click"}}'
```
