# Installing Elasticsearch

TODO: Add Security

INTRODUCE

1. Obtain Elasticsearch. See [Elastic's](https://www.elastic.co) website for a local download or [pull a docker image](https://www.docker.elastic.co/):

   ```bash
   docker pull docker.elastic.co/elasticsearch/elasticsearch:7.7.1
   ```

1. If you downloaded Elasticsearch directly, install it by extracting its archive to the system where you want it to run.

1. Install some required Elasticsearch plugins. Run these commands from the `bin` folder on each node:

   ```bash
   ./elasticsearch-plugin install analysis-icu
   ./elasticsearch-plugin install analysis-kuromoji
   ./elasticsearch-plugin install analysis-smartcn
   ./elasticsearch-plugin install analysis-stempel
   ```

   If you have a running Docker container, you can use [`docker exec -it`](https://docs.docker.com/engine/reference/commandline/exec/) to execute `elasticsearch-plugin` directives in an interactive bash shell:

   ```bash
   docker exec -it [container-name] bash -c '/usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-icu && /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-kuromoji && /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-smartcn && /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-stempel'
   ```
## Configure Elasticsearch

To configure an Elasticsearch cluster of Docker containers, use a [`docker-compose.yml` file](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docker.html#docker-compose-file). A local installation is configured via the `Elasticsearch Home/config/elasticsearch.yml` file.

```tip::
   **Docker:** For a simple single-node test cluster, you can specify the cluster name (and any other configuration options) when first creating and starting the container (see below for an example ``docker run ...`` directive).
```

These properties should have unique values for each node in the cluster:

- `node.name`
- `http.port`
- `network.host`
- `transport.port`

For example, `es-node1` might have these settings:

```yaml
node.name: es-node1
http.port: 9200
# Set this to the value declared in your etc/hosts file for the node
# TODO: Set to something unique from the node name for clarity?
network.host: es-node1
# The transport port you configure is used for discovery
transport.port: 9300
```

Depending on your system's design, every node in the Elasticsearch cluster might have identical values for these settings:

- `cluster.initial_master_nodes`
- `discovery.zen.minimum_master_nodes`
- `discovery.seed_hosts`

For example, each node of a 3-node cluster might have these settings in the `[Elasticsearch Home]/config/elasticsearch.yml` or the `docker-compose.yml`:

```yaml
cluster.initial_master_nodes:
  - es-node1
  - es-node2
  - es-node3
discovery.zen.minimum_master_nodes: 2
discovery.seed_hosts:
  - es-node1:9300
  - es-node2:9301
  - es-node3:9302
```

The `elasticsearch.yml` configuration is simpler for a single-node cluster:

```yaml
# Single node, remote elasticsearch.yml example:
discovery.type: single-node
discovery.seed_hosts:
  - es-node1:9300
http.port: 9200
network.host: es-node1
node.name: es-node1
transport.port: 9300
```

However, make sure you [force the bootstrap checks](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/bootstrap-checks.html#_forcing_the_bootstrap_checks) if you're running a single-node production server by setting a property at the end of `ES_HOME/config/jvm.options`:

```properties
# For a single node production cluster
-Des.enforce.bootstrap.checks=true
```

To configure an Elasticsearch cluster of Docker containers, the `docker-compose.yml` will contain `services` entries for each node and some common `volume` and `network` properties:

```yaml
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.7.1
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
    ulimits:
      memlock:
        soft: -1
        hard: -1
    volumes:
      - data01:/usr/share/elasticsearch/data
    ports:
      - 9200:9200
    networks:
      - elastic
  # ...
  # more nodes
  # ...
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
```

## Start Elasticsearch

If using the un-packaged Elasticsearch archive, from the `bin` folder, run 

```bash
./elasticsearch
```

If using a Docker container to drive a multi-node Elasticsearch cluster configured with a `docker-compose.yml` file, run

```bash
docker-compose up
```

A single-node test cluster can be started and configured at the same time within the `docker run` command:


```bash
docker run -p 9200:9200 -p 9300:9300 -e "cluster.name=LiferayElasticsearchCluster" -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.7.1
   ```

Elasticsearch starts, and one of its status messages includes a transport address: 

```sh
[2019-04-01T16:55:50,127][INFO ][o.e.t.TransportService   ] [HfkqdKv] publish_address {127.0.0.1:9300}, bound_addresses {[::1]:9300}, {127.0.0.1:9300}
```

Take note of this address if you're running Lideray DXP 7.2; you'll need to give it to your DXP server(s) so it can find Elasticsearch on the network. 

CONCLUDE THIS
