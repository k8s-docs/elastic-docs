Access the Elasticsearch endpointedit
You can access the Elasticsearch endpoint within or outside the Kubernetes cluster.

Within the Kubernetes cluster

Retrieve the CA certificate.
Retrieve the password of the elastic user.
NAME=hulk

kubectl get secret "$NAME-es-http-certs-public" -o go-template='{{index .data "tls.crt" | base64decode }}' > tls.crt
PW=$(kubectl get secret "\$NAME-es-elastic-user" -o go-template='{{.data.elastic | base64decode }}')

curl --cacert tls.crt -u elastic:$PW https://$NAME-es-http:9200/
Outside the Kubernetes cluster

Retrieve the CA certificate.
Retrieve the password of the elastic user.
Retrieve the IP of the LoadBalancer Service.
NAME=hulk

kubectl get secret "$NAME-es-http-certs-public" -o go-template='{{index .data "tls.crt" | base64decode }}' > tls.crt
IP=$(kubectl get svc "$NAME-es-http" -o jsonpath='{.status.loadBalancer.ingress[].ip}')
PW=$(kubectl get secret "\$NAME-es-elastic-user" -o go-template='{{.data.elastic | base64decode }}')

curl --cacert tls.crt -u elastic:$PW https://$IP:9200/
Now you should get this message:

curl: (51) SSL: no alternative certificate subject name matches target host name '35.198.131.115'
As the certificate subject by default is service name we can resolve this by using the service name and specify the resolve information through an additional argument.

> curl --cacert tls.crt -u elastic:$PW https://$NAME-es-http:9200/ --resolve $NAME-es-http:9200:$IP
> {
> "name" : "hulk-es-4qk62zd928",
> "cluster_name" : "hulk",
> "cluster_uuid" : "q6itjqFqRqW576FXF0uohg",
> "version" : {...},
> "tagline" : "You Know, for Search"
> }
> As an alternative, you can edit the spec of the Elasticsearch resource to include the external IP in certificate subject alternative names (SANs) as described below. This is useful for other clients where resolve information cannot be specified, or when it is more convenient to make changes on the server rather than client.

spec:
http:
service:
spec:
type: LoadBalancer
tls:
selfSignedCertificate:
subjectAltNames: - ip: 35.198.131.115
You can now reach Elasticsearch without --resolve:

> curl --cacert tls.crt -u elastic:$PW https://$IP:9200/
> {
> "name" : "hulk-es-4qk62zd928",
> "cluster_name" : "hulk",
> "cluster_uuid" : "q6itjqFqRqW576FXF0uohg",
> "version" : {...},
> "tagline" : "You Know, for Search"
> }
