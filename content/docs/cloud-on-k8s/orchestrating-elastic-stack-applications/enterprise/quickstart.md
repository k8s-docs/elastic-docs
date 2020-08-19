---
title: "快速开始"
linkTitle: ""
weight: 1
---

Workplace Search requires an Enterprise license on ECK. Give it a try with a trial license.

Apply the following specification to deploy Enterprise Search. ECK automatically configures the secured connection to an Elasticsearch cluster named quickstart, created in Elasticsearch quickstart.

```sh
cat <<EOF | kubectl apply -f -
apiVersion: enterprisesearch.k8s.elastic.co/v1beta1
kind: EnterpriseSearch
metadata:
name: enterprise-search-quickstart
spec:
version: 7.8.1
count: 1
elasticsearchRef:
name: quickstart
EOF
```

Monitor the Enterprise Search deployment.

Retrieve details about the Enterprise Search deployment:

```sh
kubectl get enterprisesearch
NAME HEALTH NODES VERSION AGE
enterprise-search-quickstart green 1 7.8.1 8m
List all the Pods belonging to a given deployment:
```

```sh
kubectl get pods --selector='enterprisesearch.k8s.elastic.co/name=enterprise-search-quickstart'
NAME READY STATUS RESTARTS AGE
enterprise-search-quickstart-ent-58b84db85-dl7c6 1/1 Running 0 2m50s
```

Access logs for that Pod:

kubectl logs -f enterprise-search-quickstart-ent-58b84db85-dl7c6
Access Enterprise Search

A ClusterIP Service is automatically created for the deployment, and can be used to access the Enterprise Search API from within the Kubernetes cluster:

kubectl get service enterprise-search-quickstart-ent-http
Use kubectl port-forward to access Enterprise Search from your local workstation:

```sh
kubectl port-forward service/enterprise-search-quickstart-ent-http 3002
```

Open https://localhost:3002 in your browser.

Your browser will show a warning because the self-signed certificate configured by default is not verified by a known certificate authority and not trusted by your browser. Acknowledge the warning for the purposes of this quickstart but it is highly recommended that you configure valid certificates for any production deployments.

Login as the elastic user created with the Elasticsearch cluster. Its password can be obtained with:

```sh
kubectl get secret quickstart-es-elastic-user -o=jsonpath='{.data.elastic}' | base64 --decode; echo
```
