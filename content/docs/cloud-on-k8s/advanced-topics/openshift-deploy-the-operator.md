Deploy the operatoredit
Apply the all-in-one template, as described in the quickstart.

oc apply -f https://download.elastic.co/downloads/eck/1.2.0/all-in-one.yaml
[Optional] If the Software Defined Network is configured with the ovs-multitenant plug-in, you must allow the elastic-system namespace to access other Pods and Services in the cluster:

oc adm pod-network make-projects-global elastic-system
Create a namespace to hold the Elastic resources (Elasticsearch, Kibana):

oc new-project elastic # creates the elastic project
[Optional] Allow another user or a group of users to manage the Elastic resources:

oc adm policy add-role-to-user elastic-operator developer -n elastic
In the example above the user developer is allowed to manage Elastic resources in the namespace elastic.
