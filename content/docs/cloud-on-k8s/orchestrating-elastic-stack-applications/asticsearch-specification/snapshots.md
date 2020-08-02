---
title: "创建自动快照"
linkTitle: "自动快照"
weight: 15
---

To set up automated snapshots for Elasticsearch on Kubernetes you have to:

Ensure you have the necessary Elasticsearch storage plugin installed.
Add snapshot repository credentials to the Elasticsearch keystore.
Register the snapshot repository with the Elasticsearch API.
Set up a CronJob to take snapshots on a schedule.
The examples below use the Google Cloud Storage Repository Plugin.

For more information on Elasticsearch snapshots, see Snapshot and Restore.

Install the storage repository pluginedit
To install the storage repository plugin, you can either use a custom image or add your own init container which installs the plugin when the Pod is created.

To use your own custom image with all necessary plugins pre-installed, use an Elasticsearch resource like the following one:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch-sample
spec:
  version: 7.8.1
  image: your/custom/image:tag
  nodeSets:
    - name: default
      count: 1
```

Alternatively, install the plugin when the Pod is created by using an init container:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch-sample
spec:
  version: 7.8.1
  nodeSets:
    - name: default
      count: 1
      podTemplate:
        spec:
          initContainers:
            - name: install-plugins
              command:
                - sh
                - -c
                - |
                  bin/elasticsearch-plugin install --batch repository-gcs
```

Assuming you stored this in a file called elasticsearch.yaml you can in both cases create the Elasticsearch cluster with:

kubectl apply -f elasticsearch.yaml
Configure GCS credentials via the Elasticsearch keystoreedit
The Elasticsearch GCS repository plugin requires a JSON file that contains service account credentials. These need to be added as secure settings to the Elasticsearch keystore. For more details, see Google Cloud Storage Repository Plugin.

Using ECK, you can automatically inject secure settings into a cluster node by providing them through a secret in the Elasticsearch Spec.

Create a file containing the GCS credentials. For this example, name it gcs.client.default.credentials_file. The file name is important as it is reflected in the secure setting.

```yaml
{
  "type": "service_account",
  "project_id": "your-project-id",
  "private_key_id": "...",
  "private_key": "-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n",
  "client_email": "service-account-for-your-repository@your-project-id.iam.gserviceaccount.com",
  "client_id": "...",
  "auth_uri": "https://accounts.google.com/o/oauth2/auth",
  "token_uri": "https://accounts.google.com/o/oauth2/token",
  "auth_provider_x509_cert_url": "https://www.googleapis.com/oauth2/v1/certs",
  "client_x509_cert_url": "https://www.googleapis.com/robot/v1/metadata/x509/your-bucket@your-project-id.iam.gserviceaccount.com",
}
```

Create a Kubernetes secret from that file:

```yaml
kubectl create secret generic gcs-credentials --from-file=gcs.client.default.credentials_file
```

Edit the secureSettings section of the Elasticsearch resource:

```yaml
apiVersion: elasticsearch.k8s.elastic.co/v1
kind: Elasticsearch
metadata:
  name: elasticsearch-sample
spec:
  version: 7.8.1
  # Inject secure settings into Elasticsearch nodes from a k8s secret reference
  secureSettings:
    - secretName: gcs-credentials
```

If you did not follow the instructions above and named your GCS credentials file differently, you can still map it to the expected name now. See Secure Settings for details.

Apply the modifications:

kubectl apply -f elasticsearch.yml
GCS credentials are automatically propagated into each Elasticsearch node’s keystore. It can take up to a few minutes, depending on the number of secrets in the keystore. You don’t have to restart the nodes.

Register the repository in Elasticsearchedit
Create the GCS snapshot repository in Elasticsearch. You can either use the Snapshot and Restore UI in Kibana (in version >= 7.4.0) or follow the procedure described in Snapshot and Restore:

```yaml
PUT /_snapshot/my_gcs_repository
{
  "type": "gcs",
  "settings": {
    "bucket": "my_bucket",
    "client": "default"
  }
}
```

Take a snapshot with the following HTTP request:

PUT /\_snapshot/my_gcs_repository/test-snapshot
Periodic snapshots with Snapshot Lifecycle Managementedit
You can use the snapshot lifecycle management APIs to manage policies for the time and frequency of automatic snapshots (in versions >= 7.4.0).

The Snapshot and Restore UI allows you to manage these policies directly in Kibana as well.

Periodic snapshots with a CronJobedit
If you are running older versions of Elasticsearch without the snapshot lifecycle management feature, you can still set up a simple CronJob to take a snapshot every day.

Make an HTTP request against the appropriate endpoint, using a daily snapshot naming format. Elasticsearch credentials are mounted as a volume into the job’s Pod:

```yaml
# snapshotter.yml
apiVersion: batch/v1beta1
kind: CronJob
metadata:
  name: elasticsearch-sample-snapshotter
spec:
  schedule: "@daily"
  concurrencyPolicy: Forbid
  jobTemplate:
    spec:
      template:
        spec:
          containers:
            - name: snapshotter
              image: centos:7
              volumeMounts:
                - name: es-basic-auth
                  mountPath: /mnt/elastic/es-basic-auth
              command:
                - /bin/bash
              args:
                - -c
                - 'curl -s -i -k -u "elastic:$(</mnt/elastic/es-basic-auth/elastic)" -XPUT "https://elasticsearch-sample-es-http:9200/_snapshot/my_gcs_repository/%3Csnapshot-%7Bnow%2Fd%7D%3E" | tee /dev/stderr | grep "200 OK"'
          restartPolicy: OnFailure
          volumes:
            - name: es-basic-auth
              secret:
                secretName: elasticsearch-sample-elastic-user
```

Apply it to the Kubernetes cluster:

kubectl apply -f snapshotter.yml
For more details see Kubernetes CronJobs.
