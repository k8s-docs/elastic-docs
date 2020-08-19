---
title: "common.k8s.elastic.co/v1"
linkTitle: ""
weight: 1
---

Package v1 contains API schema definitions for common types used by all resources.

## Config

Config represents untyped YAML configuration.

Appears In:

ApmServerSpec
BeatSpec
EnterpriseSearchSpec
KibanaSpec
NodeSet

## ConfigSource

ConfigSource references configuration settings.

Appears In:

BeatSpec
EnterpriseSearchSpec

Field Description
SecretRef SecretRef SecretName references a Kubernetes Secret in the same namespace as the resource that will consume it. Examples: --- # Filebeat configuration kind: Secret apiVersion: v1 metadata: name: filebeat-user-config stringData: beat.yml:

filebeat.inputs: - type: container paths: - /var/log/containers/\*.log processors: - add_kubernetes_metadata: host: \${NODE_NAME} matchers: - logs_path: logs_path: "/var/log/containers/" processors: - add_cloud_metadata: {} - add_host_metadata: {} --- # EnterpriseSearch configuration kind: Secret apiVersion: v1 metadata: name: smtp-credentials stringData: enterprise-search.yml:
email.account.enabled: true email.account.smtp.auth: plain email.account.smtp.starttls.enable: false email.account.smtp.host: 127.0.0.1 email.account.smtp.port: 25 email.account.smtp.user: myuser email.account.smtp.password: mypassword email.account.email_defaults.from: my@email.com ---

## HTTPConfig

HTTPConfig holds the HTTP layer configuration for resources.

Appears In:

ApmServerSpec
ElasticsearchSpec
EnterpriseSearchSpec
KibanaSpec
Field Description
service ServiceTemplate

Service defines the template for the associated Kubernetes Service object.

tls TLSOptions

TLS defines options for configuring TLS for HTTP.

## KeyToPath

Appears In:

SecretSource
Field Description
key string

Key is the key contained in the secret.

path string

Path is the relative file path to map the key to. Path must not be an absolute file path and must not contain any ".." components.

## ObjectSelector

ObjectSelector defines a reference to a Kubernetes object.

Appears In:

ApmServerSpec
BeatSpec
EnterpriseSearchSpec
KibanaSpec
RemoteCluster
Field Description
name string

Name of the Kubernetes object.

namespace string

Namespace of the Kubernetes object. If empty, defaults to the current namespace.

## PodDisruptionBudgetTemplate

PodDisruptionBudgetTemplate defines the template for creating a PodDisruptionBudget.

Appears In:

ElasticsearchSpec
Field Description
metadata ObjectMeta

Refer to Kubernetes API documentation for fields of metadata.

spec PodDisruptionBudgetSpec

Spec is the specification of the PDB.

## SecretRef

SecretRef is a reference to a secret that exists in the same namespace.

Appears In:

ConfigSource
FileRealmSource
RoleSource
TLSOptions
Field Description
secretName string

SecretName is the name of the secret.

## SecretSource

SecretSource defines a data source based on a Kubernetes Secret.

Appears In:

ApmServerSpec
BeatSpec
ElasticsearchSpec
KibanaSpec
Field Description
secretName string

SecretName is the name of the secret.

entries KeyToPath array

Entries define how to project each key-value pair in the secret to filesystem paths. If not defined, all keys will be projected to similarly named paths in the filesystem. If defined, only the specified keys will be projected to the corresponding paths.

## SelfSignedCertificate

Appears In: TLSOptions

| Field           |                              | Description                                                                                 |
| --------------- | ---------------------------- | ------------------------------------------------------------------------------------------- |
| subjectAltNames | SubjectAlternativeName array | SubjectAlternativeNames is a list of SANs to include in the generated HTTP TLS certificate. |
| disabled        | boolean                      | Disabled indicates that the provisioning of the self-signed certifcate should be disabled.  |

## ServiceTemplate

ServiceTemplate defines the template for a Kubernetes Service.

Appears In:

HTTPConfig
TransportConfig

| Field    |             | Description                                                   |
| -------- | ----------- | ------------------------------------------------------------- |
| metadata | ObjectMeta  | Refer to Kubernetes API documentation for fields of metadata. |
| spec     | ServiceSpec | Spec is the specification of the service.                     |

## SubjectAlternativeName

Appears In:

SelfSignedCertificate

| Field |        | Description                          |
| ----- | ------ | ------------------------------------ |
| dns   | string | DNS is the DNS name of the subject.  |
| ip    | string | IP is the IP address of the subject. |

## TLSOptions

TLSOptions holds TLS configuration options.

Appears In:

HTTPConfig

| Field                 | Type                  | Description                                                                                                                                                                                                                                                                                                                                  |
| --------------------- | --------------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| selfSignedCertificate | SelfSignedCertificate | SelfSignedCertificate allows configuring the self-signed certificate generated by the operator.                                                                                                                                                                                                                                              |
| certificate           | SecretRef             | Certificate is a reference to a Kubernetes secret that contains the certificate and private key for enabling TLS. The referenced secret should contain the following: - ca.crt: The certificate authority (optional). - tls.crt: The certificate (or a chain). - tls.key: The private key to the first certificate in the certificate chain. |
