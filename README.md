# Automatizar

Automatizar contains docker-compose and automation scripts for deploying colocard and supporting services in a production environment.

## Project Organization

Production colocard consists of four sets of services deployed in docker containers using docker-compose.

1. colocard service set
    1. colocard web server
    2. keycloak-proxy.js server (authentication)
    3. mongodb database
    4. mongo-grove (automated mongodb backup)
2. elastic stack service set
    1. elasticsearch
    2. logstash
    3. kibana
3. nginx reverse proxy service set
    1. nginx web server
    2. docker-gen
    ~~3. letsencrypt-nginx-proxy-companion~~
4. elastic beats service set
    1. filebeat

### colocard Service Set

Project location: `colocard/`

The colocard service set is defined in the `docker-compose.yml` file under the `colocard` directory. This service set is responsible for running the colocard API server and associated services, including a mongodb database, a Keycloak authentication proxy using keycloak-proxy.js, and mongo-grove, an automated mongodb backup service. All services run on the colocard network, which must be created before starting the services for the first time (detailed below).

All configuration files are stored under the `configs` directory. Currently, this includes configuration settings for the colocard API server and keycloak-proxy.js.

### Elastic Stack Service Set

Project location: `elk/`

The elastic stack service set is defined in the `docker-compose.yml` file under the `elk` directory. This service set is responsible for collecting, transforming, storing, and indexing logs produced by the colocard and nginx service sets using the elastic stack: elasticsearch, logstash, and kibana. All services run on the elk network, which must be created before starting the services for the first time (detailed below).

All configuration files are stored under the `configs` directory. Currently, this includes the logstash pipeline configuration.

### NGINX Service Set

REMOVED. Now using:

https://github.com/evertramos/nginx-proxy-automation

### Elastic Beats Service Set

Project location: `beats/`

The elastic beats service set is defined in the `docker-compose.yml` file under the `beats` directory. This service set is responsible for collecting and sending logs (and potentially other information in the future) to the elastic stack service set. Currently, the only service that runs is Filebeat. This service set connects to both the colocard and elk networks.

## Getting Started

First, create the following Docker networks.

```sh
docker network create neuvue
docker network create elk
```

We create custom networks as opposed to letting docker-compose do it for us due to the fact that we'll have four service sets running out of different directories that need to communicate with each other.

Next, we'll begin launching the services. Theoretically, the order the are started in shouldn't matter; however, to be on the safe side, launch them in the following order:

1. elastic stack service set
2. neuvue service set.
4. elastic beats service set.

All services sets can be started by issuing the following command in their respective project directories:

```sh
docker-compose up -d
```

### Elastic Stack Service Set First-time Setup

When you launch the elastic stack service set for the first time, several commands must be run to initialize elasticsearch and kibana to optimally work with the beats defined in the elastic beats service stack. Run the following commands:

```sh
docker run --rm --network=elk docker.elastic.co/beats/filebeat:7.7.0 setup --index-management \
    -E output.logstash.enabled=false \
    -E 'output.elasticsearch.hosts=["elasticsearch:9200"]'
docker run --rm --network=elk docker.elastic.co/beats/filebeat:7.7.0 setup -e \
    -E output.logstash.enabled=false \
    -E 'output.elasticsearch.hosts=["elasticsearch:9200"]' \
    -E setup.kibana.host="kibana:5601"
```

### NGINX Service Set First-time Setup

colocard responses can have very large responses when a user submits a large query. By default, nginx's response buffer size is not large enough to handle queries analysts may sometimes run. Included in the `configs` directory under the nginx project directory is a conf file which greatly increases the default buffer sizes for nginx. Currently, this file must be manually copied to the appropriate directory. Once the nginx service set has been launched, copy the `buffers.conf` file to `/etc/nginx/conf.d` directory in the nginx container.

```sh
docker cp configs/buffers.conf nginx-web:/etc/nginx/conf.d/buffers.conf
# for good measure
docker exec nginx-web chown root:root /etc/nginx/conf.d/buffers.conf
docker exec nginx-web chmod 0644 /etc/nginx/conf.d/buffers.conf
```

Once you have copied the config file, restart the nginx container.

```sh
docker-compose restart nginx-web
```
