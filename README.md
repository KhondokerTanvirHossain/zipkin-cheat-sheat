# Elasticsearch and Zipkin Docker Compose Setup

This repository contains a Docker Compose setup for running Elasticsearch and Zipkin with data persistence and snapshot backup configuration.

## Prerequisites

- Docker
- Docker Compose

## Setup

### Step 1: Create Directories

Create directories on your host machine to store Elasticsearch data and snapshots:

```sh
mkdir -p /path/to/your/snapshot/repo
mkdir -p /path/to/your/elasticsearch/data
```

### Step 2: Create elasticsearch.yml

Create a file named `elasticsearch.yml` with the following content:

```yml
# Elasticsearch configuration

# Cluster name
cluster.name: "docker-cluster"

# Node name
node.name: "es01"

# Path to directory where to store the data (separate multiple locations by comma):
path.data: /usr/share/elasticsearch/data

# Path to log files:
path.logs: /usr/share/elasticsearch/logs

# Path to snapshot repository
path.repo: ["/usr/share/elasticsearch/snapshots"]

# Network settings
network.host: 0.0.0.0

# Discovery settings
discovery.type: single-node

# X-Pack security settings (optional, if you are using X-Pack)
# xpack.security.enabled: true
# xpack.security.transport.ssl.enabled: true
# xpack.security.http.ssl.enabled: true

# JVM options (optional, if you want to set JVM options here instead of using ES_JAVA_OPTS)
# jvm.options:
#   -Xms512m
#   -Xmx512m
```

### Step 3: Create docker-compose.yml

Create a file named `docker-compose.yml` with the following content:

```yml
version: '3.8'

services:
  elasticsearch:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.17.0
    container_name: es01-test
    environment:
      - discovery.type=single-node
      - ES_JAVA_OPTS=-Xms512m -Xmx512m
    ports:
      - "9200:9200"
      - "9300:9300"
    volumes:
      - esdata:/usr/share/elasticsearch/data
      - /path/to/your/snapshot/repo:/usr/share/elasticsearch/snapshots
      - /path/to/your/elasticsearch.yml:/usr/share/elasticsearch/config/elasticsearch.yml

  zipkin:
    image: openzipkin/zipkin
    container_name: zipkin
    environment:
      - STORAGE_TYPE=elasticsearch
      - ES_HOSTS=http://elasticsearch:9200
    ports:
      - "9411:9411"
    depends_on:
      - elasticsearch

volumes:
  esdata:
    driver: local
```

Replace `/path/to/your/snapshot/repo` and `/path/to/your/elasticsearch.yml` with the actual paths on your host machine.

### Step 4: Start the Services

Run the following command to start the services:

```sh
docker-compose up -d
```

### Step 5: Register the Snapshot Repository

Register the snapshot repository with Elasticsearch:

```sh
curl -X PUT "localhost:9200/_snapshot/my_backup" -H 'Content-Type: application/json' -d'
{
  "type": "fs",
  "settings": {
    "location": "/usr/share/elasticsearch/snapshots",
    "compress": true
  }
}
'
```

### Step 6: Create a Snapshot

Create a snapshot of your Elasticsearch indices:

### Step 7: Restore a Snapshot

To restore a snapshot, use the following command:

```sh
curl -X POST "localhost:9200/_snapshot/my_backup/snapshot_1/_restore"
```

### Notes

- Adjust the memory settings in ES_JAVA_OPTS as needed based on the available resources and the expected workload.
- Ensure that the paths specified in the docker-compose.yml file match the actual paths on your host machine.
- This setup ensures that Elasticsearch and Zipkin are configured with data persistence and snapshot backup capabilities.