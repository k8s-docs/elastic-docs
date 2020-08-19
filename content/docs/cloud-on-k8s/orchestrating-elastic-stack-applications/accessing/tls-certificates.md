TLS Certificatesedit
This section only covers TLS certificates for the HTTP layer. Those for the transport layer used for Elasticsearch internal communication between Elasticsearch nodes in a cluster are managed by ECK and are not configurable.

Default self-signed certificateedit
By default, the operator manages a self-signed certificate with a custom CA for each resource. The CA, the certificate and the private key are each stored in a separate Secret.

> kubectl get secret | grep es-http
> hulk-es-http-ca-internal Opaque 2 28m
> hulk-es-http-certs-internal Opaque 2 28m
> hulk-es-http-certs-public Opaque 1 28m
> The public certificate is stored in a secret named <name>-[es|kb|apm|ent]-http-certs-public.

> kubectl get secret hulk-es-http-certs-public -o go-template='{{index .data "tls.crt" | base64decode }}'
> -----BEGIN CERTIFICATE-----
> MIIDQDCCAiigAwIBAgIQHC4O/RWX15a3/P3upsm3djANBgkqhkiG9w0BAQsFADA6
> ...
> QLYL4zLEby3vRxq65+xofVBJAaM=
> -----END CERTIFICATE-----
> Reserve static IP and custom domainedit
> To use a custom domain name with the self-signed certificate, you can reserve a static IP and/or use an Ingress instead of a LoadBalancer Service. Whatever you use, your DNS must be added to the certificate SAN in the spec.http.tls.selfSignedCertificate.subjectAltNames section of your Elastic resource manifest.

spec:
http:
service:
spec:
type: LoadBalancer
tls:
selfSignedCertificate:
subjectAltNames: - ip: 160.46.176.15 - dns: hulk.example.com
Setup your own certificateedit
You can bring your own certificate to configure TLS to ensure that communication between HTTP clients and the cluster is encrypted.

Create a Kubernetes secret with:

ca.crt: CA certificate (optional if tls.crt was issued by a well-known CA).
tls.crt: the certificate.
tls.key: the private key to the first certificate in the certificate chain.
kubectl create secret generic my-cert --from-file=ca.crt=tls.crt --from-file=tls.crt=tls.crt --from-file=tls.key=tls.key
Then you just have to reference the secret name in the http.tls.certificate section of the resource manifest.

spec:
http:
tls:
certificate:
secretName: my-cert
Disable TLSedit
You can explicitly disable TLS for Kibana, APM Server, Enterprise Search and the HTTP layer of Elasticsearch.

spec:
http:
tls:
selfSignedCertificate:
disabled: true
