Securityedit
All Elastic Stack resources deployed by the ECK Operator are secured by default. The operator sets up basic authentication and TLS to encrypt network traffic to, from, and within your Elasticsearch cluster.

Authenticationedit
To access Elastic resources, the operator manages a default user named elastic with the superuser role. Its password is stored in a Secret named <name>-elastic-user.

> kubectl get secret hulk-es-elastic-user -o go-template='{{.data.elastic | base64decode }}'
> 42xyz42citsale42xyz42
> Beware of copying this Secret as-is into a different namespace. See Common Problems: Owner References for more information.
