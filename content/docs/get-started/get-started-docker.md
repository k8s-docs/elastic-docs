---
title: "在Docker上运行弹性堆栈"
linkTitle: ""
weight: 2
---

The Elastic Docker registry contains Docker images for all the products in the Elastic Stack: https://www.docker.elastic.co/.

Run with Docker Composeedit
To get the default distributions of Elasticsearch and Kibana up and running in Docker, you can use Docker Compose.

Create a docker-compose.yml file for the Elastic Stack. The following example brings up a three node cluster and Kibana so you can see how things work. This all-in-one configuration is a handy way to bring up your first dev cluster before you build a distributed deployment with multiple hosts.

version: '2.2'
services:
es01:
image: docker.elastic.co/elasticsearch/elasticsearch:7.8.1
container_name: es01
environment: - node.name=es01 - cluster.name=es-docker-cluster - discovery.seed_hosts=es02,es03 - cluster.initial_master_nodes=es01,es02,es03 - bootstrap.memory_lock=true - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
ulimits:
memlock:
soft: -1
hard: -1
volumes: - data01:/usr/share/elasticsearch/data
ports: - 9200:9200
networks: - elastic

es02:
image: docker.elastic.co/elasticsearch/elasticsearch:7.8.1
container_name: es02
environment: - node.name=es02 - cluster.name=es-docker-cluster - discovery.seed_hosts=es01,es03 - cluster.initial_master_nodes=es01,es02,es03 - bootstrap.memory_lock=true - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
ulimits:
memlock:
soft: -1
hard: -1
volumes: - data02:/usr/share/elasticsearch/data
ports: - 9201:9200
networks: - elastic

es03:
image: docker.elastic.co/elasticsearch/elasticsearch:7.8.1
container_name: es03
environment: - node.name=es03 - cluster.name=es-docker-cluster - discovery.seed_hosts=es01,es02 - cluster.initial_master_nodes=es01,es02,es03 - bootstrap.memory_lock=true - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
ulimits:
memlock:
soft: -1
hard: -1
volumes: - data03:/usr/share/elasticsearch/data
ports: - 9202:9200
networks: - elastic

kib01:
image: docker.elastic.co/kibana/kibana:7.8.1
container_name: kib01
ports: - 5601:5601
environment:
ELASTICSEARCH_URL: http://es01:9200
ELASTICSEARCH_HOSTS: http://es01:9200
networks: - elastic

volumes:
data01:
driver: local
data02:
driver: local
data03:
driver: local

networks:
elastic:
driver: bridge
Make sure Docker Engine is allotted at least 4GiB of memory. In Docker Desktop, you configure resource usage on the Advanced tab in Preference (macOS) or Settings (Windows).
Run docker-compose to bring up the three-node Elasticsearch cluster and Kibana:

docker-compose up
Submit a \_cat/nodes request to see that the nodes are up and running:

curl -X GET "localhost:9200/\_cat/nodes?v&pretty"
Open Kibana to load sample data and interact with the cluster: http://localhost:5601.
When you’re done experimenting, you can tear down the containers and volumes by running docker-compose down -v.

Run in Docker with TLS enablededit
If you have a Gold (or higher) subscription and the security features are enabled, you must configure Transport Layer Security (TLS) encryption for the Elasticsearch transport layer. While it is possible to use a trial license without setting up TLS, we advise securing your stack from the start.

To get an Elasticsearch cluster and Kibana up and running in Docker with security enabled, you can use Docker Compose:

Create the following compose and configuration files. These files are also available from the elastic/stack-docs repository on GitHub.

instances.yml identifies the instances you need to create certificates for.
.env sets environment variables to specify the Elasticsearch version and the location where the Elasticsearch certificates will be created.
create-certs.yml is a Docker Compose file that launches a container to generate the certificates for Elasticsearch and Kibana.
elastic-docker-tls.yml is a Docker Compose file that brings up a three-node Elasticsearch cluster and a Kibana instance with TLS enabled so you can see how things work. This all-in-one configuration is a handy way to bring up your first dev cluster before you build a distributed deployment with multiple hosts.
instances.yml:

instances:

- name: es01
  dns:

  - es01
  - localhost
    ip:
  - 127.0.0.1

- name: es02
  dns:

  - es02
  - localhost
    ip:
  - 127.0.0.1

- name: es03
  dns:

  - es03
  - localhost
    ip:
  - 127.0.0.1

- name: 'kib01'
  dns: - kib01 - localhost
  .env:

COMPOSE_PROJECT_NAME=es
CERTS_DIR=/usr/share/elasticsearch/config/certificates
VERSION=7.8.1
create-certs.yml:

version: '2.2'

services:
create_certs:
image: docker.elastic.co/elasticsearch/elasticsearch:\${VERSION}
container_name: create_certs
command: >
bash -c '
yum install -y -q -e 0 unzip;
if [[ ! -f /certs/bundle.zip ]]; then
bin/elasticsearch-certutil cert --silent --pem --in config/certificates/instances.yml -out /certs/bundle.zip;
unzip /certs/bundle.zip -d /certs;
fi;
chown -R 1000:0 /certs
'
working_dir: /usr/share/elasticsearch
volumes: - certs:/certs - .:/usr/share/elasticsearch/config/certificates
networks: - elastic

volumes:
certs:
driver: local

networks:
elastic:
driver: bridge
elastic-docker-tls.yml:

version: '2.2'

services:
es01:
image: docker.elastic.co/elasticsearch/elasticsearch:${VERSION}
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.license.self_generated.type=trial
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=$CERTS_DIR/es01/es01.key - xpack.security.http.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.http.ssl.certificate=$CERTS_DIR/es01/es01.crt - xpack.security.transport.ssl.enabled=true - xpack.security.transport.ssl.verification_mode=certificate - xpack.security.transport.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.transport.ssl.certificate=$CERTS_DIR/es01/es01.crt - xpack.security.transport.ssl.key=$CERTS_DIR/es01/es01.key
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
      - certs:$CERTS_DIR
ports: - 9200:9200
networks: - elastic

    healthcheck:
      test: curl --cacert $CERTS_DIR/ca/ca.crt -s https://localhost:9200 >/dev/null; if [[ $$? == 52 ]]; then echo 0; else echo 1; fi
      interval: 30s
      timeout: 10s
      retries: 5

es02:
image: docker.elastic.co/elasticsearch/elasticsearch:${VERSION}
    container_name: es02
    environment:
      - node.name=es02
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.license.self_generated.type=trial
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=$CERTS_DIR/es02/es02.key - xpack.security.http.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.http.ssl.certificate=$CERTS_DIR/es02/es02.crt - xpack.security.transport.ssl.enabled=true - xpack.security.transport.ssl.verification_mode=certificate - xpack.security.transport.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.transport.ssl.certificate=$CERTS_DIR/es02/es02.crt - xpack.security.transport.ssl.key=$CERTS_DIR/es02/es02.key
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data02:/usr/share/elasticsearch/data
      - certs:$CERTS_DIR
networks: - elastic

es03:
image: docker.elastic.co/elasticsearch/elasticsearch:${VERSION}
    container_name: es03
    environment:
      - node.name=es03
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es01,es02
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      - xpack.license.self_generated.type=trial
      - xpack.security.enabled=true
      - xpack.security.http.ssl.enabled=true
      - xpack.security.http.ssl.key=$CERTS_DIR/es03/es03.key - xpack.security.http.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.http.ssl.certificate=$CERTS_DIR/es03/es03.crt - xpack.security.transport.ssl.enabled=true - xpack.security.transport.ssl.verification_mode=certificate - xpack.security.transport.ssl.certificate_authorities=$CERTS_DIR/ca/ca.crt
      - xpack.security.transport.ssl.certificate=$CERTS_DIR/es03/es03.crt - xpack.security.transport.ssl.key=$CERTS_DIR/es03/es03.key
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data03:/usr/share/elasticsearch/data
      - certs:$CERTS_DIR
networks: - elastic
kib01:
image: docker.elastic.co/kibana/kibana:${VERSION}
    container_name: kib01
    depends_on: {"es01": {"condition": "service_healthy"}}
    ports:
      - 5601:5601
    environment:
      SERVERNAME: localhost
      ELASTICSEARCH_URL: https://es01:9200
      ELASTICSEARCH_HOSTS: https://es01:9200
      ELASTICSEARCH_USERNAME: kibana_system
      ELASTICSEARCH_PASSWORD: CHANGEME
      ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES: $CERTS_DIR/ca/ca.crt
SERVER_SSL_ENABLED: "true"
SERVER_SSL_KEY: $CERTS_DIR/kib01/kib01.key
      SERVER_SSL_CERTIFICATE: $CERTS_DIR/kib01/kib01.crt
volumes: - certs:\$CERTS_DIR
networks: - elastic
volumes:
data01:
driver: local
data02:
driver: local
data03:
driver: local
certs:
driver: local

networks:
elastic:
driver: bridge

Generate and apply a trial license that supports Transport Layer Security.

Enable Transport Layer Security to encrypt client communications.

Enable Transport Layer Security to encrypt internode communications.

Allow the use of self-signed certificates by not requiring hostname verification.

Make sure Docker Engine is allotted at least 4GiB of memory. In Docker Desktop, you configure resource usage on the Advanced tab in Preference (macOS) or Settings (Windows).
Generate certificates for Elasticsearch by bringing up the create-certs container:

docker-compose -f create-certs.yml run --rm create_certs
Bring up the three-node Elasticsearch cluster:

docker-compose -f elastic-docker-tls.yml up -d
At this point, Kibana cannot connect to the Elasticsearch cluster. You must generate a password for the built-in kibana_system user, update the ELASTICSEARCH_PASSWORD in the compose file, and restart to enable Kibana to communicate with the secured cluster.

Run the elasticsearch-setup-passwords tool to generate passwords for all built-in users, including the kibana_system user. If you don’t use PowerShell on Windows, remove the trailing `\`characters and join the lines before running this command.

docker exec es01 /bin/bash -c "bin/elasticsearch-setup-passwords \
auto --batch --url https://es01:9200"
Make a note of the generated passwords. You must configure the kibana_system user password in the compose file to enable Kibana to connect to Elasticsearch, and you’ll need the password for the elastic superuser to log in to Kibana and submit requests to Elasticsearch.

Set ELASTICSEARCH_PASSWORD in the elastic-docker-tls.yml compose file to the password generated for the kibana_system user.

kib01:
image: docker.elastic.co/kibana/kibana:${VERSION}
    container_name: kib01
    depends_on: {"es01": {"condition": "service_healthy"}}
    ports:
      - 5601:5601
    environment:
      SERVERNAME: localhost
      ELASTICSEARCH_URL: https://es01:9200
      ELASTICSEARCH_HOSTS: https://es01:9200
      ELASTICSEARCH_USERNAME: kibana_system
      ELASTICSEARCH_PASSWORD: CHANGEME
      ELASTICSEARCH_SSL_CERTIFICATEAUTHORITIES: $CERTS_DIR/ca/ca.crt
SERVER_SSL_ENABLED: "true"
SERVER_SSL_KEY: $CERTS_DIR/kib01/kib01.key
      SERVER_SSL_CERTIFICATE: $CERTS_DIR/kib01/kib01.crt
volumes: - certs:\$CERTS_DIR
networks: - elastic
Use docker-compose to restart the cluster and Kibana:

docker-compose stop
docker-compose -f elastic-docker-tls.yml up -d
Open Kibana to load sample data and interact with the cluster: https://localhost:5601.

Because SSL is also enabled for communications between Kibana and client browsers, you must access Kibana via the HTTPS protocol.

When you’re done experimenting, you can tear down the containers, network, and volumes by running docker-compose -f elastic-docker-tls.yml down -v.

Loading settings from a fileedit
Specifying settings for Elasticsearch and {Kibana} directly in the compose file is a convenient way to get started, but loading settings from a file is preferable once you get past the experimental stage.

For example, to use es01.yml as the configuration file for the es01 Elasticsearch node, you can create a bind mount in the volumes section.

    volumes:
      - data01:/usr/share/elasticsearch/data
      - certs:$CERTS_DIR
      - ./es01.yml:/usr/share/elasticsearch/config/elasticsearch.yml

Similarly, to load Kibana settings from a file, you overwrite /usr/share/kibana/config/kibana.yml:

    volumes:
      - certs:$CERTS_DIR
      - ./kibana.yml:/usr/share/kibana/config/kibana.yml

Product-specific instructions for Dockeredit
See the product-specific documentation for information about running a specific Elastic product in Docker:

Install Elasticsearch with Docker
Running APM Server on Docker
Running Auditbeat on Docker
Running Filebeat on Docker
Running Heartbeat on Docker
Running Kibana on Docker
Running Logstash on Docker
Running Metricbeat on Docker
Running Packetbeat on Docker
