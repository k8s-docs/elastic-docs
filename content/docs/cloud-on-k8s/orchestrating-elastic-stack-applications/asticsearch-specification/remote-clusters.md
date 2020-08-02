---
title: "远程集群"
linkTitle: ""
weight: 16
---

The remote clusters module in Elasticsearch enables you to establish uni-directional connections to a remote cluster. This functionality is used in cross-cluster replication and cross-cluster search.

When using remote cluster connections with ECK, the setup process depends on where the remote cluster is deployed.

Connect from an Elasticsearch cluster running in the same Kubernetes clusteredit
The remote clusters feature requires a valid Enterprise license or Enterprise trial license. See the license documentation for more details about managing licenses.

You can create a remote cluster connection to another Elasticsearch cluster deployed within the same Kubernetes cluster by specifying the remoteClusters attribute in your Elasticsearch spec.

For illustration purposes, consider the following example assuming you want to configure cluster-two as a remote cluster in cluster-one.

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: cluster-one
  namespace: ns-one
spec:
  nodeSets:
    - count: 3
      name: default
  remoteClusters:
    - name: cluster-two
      elasticsearchRef:
        name: cluster-two
        namespace: ns-two
  version: 7.8.1
```

The namespace declaration can be omitted if both clusters reside in the same namespace.

Connect from an Elasticsearch cluster running outside the Kubernetes clusteredit
While it is technically possible to configure remote cluster connections using older versions of Elasticsearch, this guide only covers the setup for Elasticsearch 7.6 and later. The setup process is significantly simplified in Elasticsearch 7.6 due to improved support for the indirection of Kubernetes services.

You can configure a remote cluster connection to an ECK-managed Elasticsearch cluster from another cluster running outside the Kubernetes cluster as follows:

Ensure that both clusters trust each other’s certificate authority.
Configure the remote cluster connection via the Elasticsearch REST API.
For illustration purposes, consider the following example:

cluster-one resides inside Kubernetes and is managed by ECK
cluster-two is not hosted inside the same Kubernetes cluster as cluster-one and may not even be managed by ECK
To configure cluster-one as a remote cluster in cluster-two:

Ensure both clusters trust each others certificate authorityedit
The certificate authority (CA) used by ECK to issue certificates for the Elasticsearch transport layer is stored in a secret named <cluster_name>-es-transport-certs-public. Extract the certificate for cluster-one as follows:

kubectl get secret cluster-one-es-transport-certs-public \
-o go-template='{{index .data "ca.crt" | base64decode}}' > remote.ca.crt
You then need to configure the CA as one of the trusted CAs in cluster-two. If that cluster is hosted outside of Kubernetes, simply add the CA certificate extracted in the above step to the list of CAs in xpack.security.transport.ssl.certificate_authorities.

Beware of copying the source Secret as-is into a different namespace. See Common Problems: Owner References for more information.

CA certificates are automatically rotated after one year by default. You can configure this period. Make sure to keep the copy of the certificates Secret up-to-date.

If cluster-two is also managed by an ECK instance, proceed as follows:

Create a secret with the CA certificate you just extracted:

kubectl create secret generic remote-certs --from-file=remote.ca.crt
Use this secret to configure cluster-one's CA as a trusted CA in cluster-two:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: cluster-two
spec:
  nodeSets:
    - config:
        xpack.security.transport.ssl.certificate_authorities:
          - /usr/share/elasticsearch/config/other/remote.ca.crt
      count: 3
      name: default
      podTemplate:
        spec:
          containers:
            - name: elasticsearch
              volumeMounts:
                - mountPath: /usr/share/elasticsearch/config/other
                  name: remote-certs
          volumes:
            - name: remote-certs
              secret:
                secretName: remote-certs
  version: 7.8.1
```

Repeat the above steps to add the CA of cluster-two to cluster-one as well.

Configure the remote cluster connection via the Elasticsearch REST APIedit
Expose the transport layer of cluster-one.

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: cluster-one
spec:
  transport:
    service:
      spec:
        type: LoadBalancer
```

On cloud providers which support external load balancers, setting the type field to LoadBalancer provisions a load balancer for your Service. Alternatively expose the service via one of the Kubernetes Ingress controllers that support TCP services.

Finally, configure cluster-one as a remote cluster in cluster-two using the Elasticsearch REST API:

```yaml
PUT _cluster/settings
{
  "persistent": {
    "cluster": {
      "remote": {
        "cluster-one": {
          "mode": "proxy",
          "proxy_address": "${LOADBALANCER_IP}:9300"
        }
      }
    }
  }
}
```

Use "proxy" mode as cluster-two will be connecting to cluster-one through the Kubernetes service abstraction.

Replace \${LOADBALANCER_IP} with the IP address assigned to the LoadBalancer configured above. if you have configured a DNS entry for the service, you can use the DNS name instead of the IP address as well.
