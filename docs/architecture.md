# colocard Production Architecture

## Overview

As mentioned in the main [README](../README.md), production colocard consists for four service sets. The diagram below details the exact architecture used for colocard. The NGINX service set sits in front of everything, acting as a proxy to other services. Amongst all service sets, only the NGINX web server exposes a port via the host; this is done for security purposes. The two other docker containers in the NGINX server set configure NGINX's proxy and HTTPS settings automatically based on configuration files and docker-compose environment variables. HTTP traffic is proxied by the NGINX server to elk and colocard service sets, specifically the Kibana and keycloak-proxy.js containers. Both of these service sets reside on their own docker network, which the NGINX service set can access.

The colocard service set is composed off the various applications to run the colocard API, including the Keycloak security proxy, main API server, MongoDB database, and database backup. keycloak-proxy.js, the Keycloak security proxy, ensures that only authorized users are able to interact with the colocard API. All containers in this service set are connected to the colocard docker network so that they can communicate with each other.

The elk service set runs the Elastic Stack, specifically Kibana, Elasticsearch, and Logstash. This service set aggregates, transforms, stores, and indexes various logs produced by the NGINX and colocard service sets. All containers in this service set are connected to the elk docker network so that they can communicate with each other.

Lastly, the beats service stack collects logs via various Elastic Beats. Currently, this only includes Filebeat. Like the NGINX service set, the beats service stacks connects to both docker networks.

## Architecture Diagram

![colocard Architecture](img/colocard-architecture.jpg?raw=true)

