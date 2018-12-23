# automatizar

colocard automation scripts

## Getting Started

First, create the following Docker networks.

```sh
docker network create colocard
docker network create elk
```

TODO

### Elastic Stack First-time Setup

TODO

```sh
docker run --rm --network=elk docker.elastic.co/beats/filebeat:6.5.2 setup --template \
    -E output.logstash.enabled=false \
    -E 'output.elasticsearch.hosts=["elasticsearch:9200"]'
docker run --rm --network=elk docker.elastic.co/beats/filebeat:6.5.2 setup -e \
    -E output.logstash.enabled=false \
    -E 'output.elasticsearch.hosts=["elasticsearch:9200"]' \
    -E setup.kibana.host=kibana:5601
```
