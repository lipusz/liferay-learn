# Installing Elasticsearch

<!-- TODO: Add Security -->

Once your [system is ready](./getting-started-with-elasticsearch.md), install and configure the Elasticsearch cluster that you'll later [connect to Liferay CE/DXP](./connecting-to-elasticsearch.md).

1. Obtain Elasticsearch. See [Elastic's](https://www.elastic.co) website for a local download or [pull a docker image](https://www.docker.elastic.co/):

   ```bash
   docker pull docker.elastic.co/elasticsearch/elasticsearch:7.9.0
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

1. Generate the certificates and keys needed to securely connect to Elasticsearch. Using the [elasticsearch-util](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/certutil.html) command simplifies the process.

   If you'll be running a Docker container, follow Elastic's [Encrypting communications in an Elasticsearch Docker Container](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/configuring-tls-docker.html).
<!-- not about cert format here; pem needed for kibana -->
<!-- provide the commands here too-->

## Configure Elasticsearch

The Elasticsearch configuration is an important step in setting up Elasticsearch for Liferay DXP. Before configuring anything, consider these important points:

- Security (authentication and encryption of the Elasticsearch communication between nodes, and to clients) should always be enabled in production.
- PEM certificates are required if you also want to use Kibana with Liferay DXP and Elasticsearch.
<!-- Anything else to point out here? -->

### An Example On-Premise Configuration File

An on-premise Elasticsearch cluster is primarily configured in the `Elasticsearch Home/config/elasticsearch.yml` files of each node.  

The `elasticsearch.yml` for a node `es-node3` that's part of a three-node cluster called `LiferayElasticsearchCluster` might have these settings:

```yaml
cluster.name: LiferayElasticsearchCluster

# X-Pack Security
xpack.security.enabled: true

## TLS/SSL settings for Transport layer
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.key: certs/elastic-certificates.key
xpack.security.transport.ssl.certificate: certs/elastic-certificates.crt
xpack.security.transport.ssl.certificate_authorities : ["certs/ca.crt"]

# TLS/SSL settings for HTTP layer
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.verification_mode: certificate
xpack.security.http.ssl.key: certs/elastic-certificates.key
xpack.security.http.ssl.certificate: certs/elastic-certificates.crt
xpack.security.http.ssl.certificate_authorities : ["certs/ca.crt"]

# Remote prod
cluster.initial_master_nodes:
  - es-node1
  - es-node2
  - es-node3
discovery.seed_hosts:
  - es-node1:9300
  - es-node2:9301
  - es-node3:9302
http.port: 9202
network.host: es-node3
node.name: es-node3
transport.port: 9302
```

> **Related Elasticsearch Documentation:** 
> - [Important Elasticsearch configuration](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/important-settings.html)
> - [Security settings in Elasticsearch](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/security-settings.html)

These properties should have unique values for each node in the cluster:

- `node.name`
- `http.port`
- `network.host`
- `transport.port`

For example, the first node, `es-node1` might have these settings:

```yaml
node.name: es-node1
http.port: 9200
# Set this to the value declared in your etc/hosts file for the node
# TODO: Set to something unique from the node name for clarity?
network.host: es-node1
# The transport port you configure is used for discovery
transport.port: 9300
...
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

This is a fine configuration for development and testing that relies on a remote Elasticsearch server. For production, you should [force the bootstrap checks](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/bootstrap-checks.html#_forcing_the_bootstrap_checks) and always configure security (authentication and encryption).

To force the bootstrap check, add a property to the end of `ES_HOME/config/jvm.options`:

```properties
# For a single node production cluster
-Des.enforce.bootstrap.checks=true
```
### Configuring Elasticsearch for Docker

To configure an Elasticsearch cluster of Docker containers, use these files:

- `instances.yml`
- `.env`
- `docker-compose.yml`
- `create-certs.yml`

> **Related Elasticsearch Documentation:** 
> - [Install Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/docker.html)
> - [Encrypt Elasticsearch with Docker](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/configuring-tls-docker.html)

```tip::
   **Docker:** For a simple single-node test cluster, you can specify the cluster name (and any other configuration options) when first creating and starting the container (see `Start Elasticsearch`_ for an example ``docker run ...`` directive).
```

The `docker-compose.yml` will contain `services` entries for each node and some common `volume` and `network` properties:

```yaml
version: '2.2'
services:
  es01:
    image: docker.elastic.co/elasticsearch/elasticsearch:7.9.0
    container_name: es01
    environment:
      - node.name=es01
      - cluster.name=es-docker-cluster
      - discovery.seed_hosts=es02,es03
      - cluster.initial_master_nodes=es01,es02,es03
      - bootstrap.memory_lock=true
      - "ES_JAVA_OPTS=-Xms512m -Xmx512m"
      # include any additional settings, like ssl and http security
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
<!-- add security settings-->

## Start Elasticsearch

If using the unpackaged Elasticsearch archive, from the `bin` folder, run 

```bash
./elasticsearch
```

If using a Docker container to drive a multi-node Elasticsearch cluster configured with a `docker-compose.yml` file, run

```bash
docker-compose up
```

A single-node test cluster can be started and configured at the same time within the `docker run` command:

```bash
docker run -p 9200:9200 -p 9300:9300 -e "cluster.name=LiferayElasticsearchCluster" -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.9.0
   ```

Elasticsearch starts, and one of its status messages includes a transport address: 

```sh
[2019-04-01T16:55:50,127][INFO ][o.e.t.TransportService   ] [HfkqdKv] publish_address {127.0.0.1:9300}, bound_addresses {[::1]:9300}, {127.0.0.1:9300}
```

Take note of this address if you're running Liferay DXP 7.2; you'll need to give it to your DXP server(s) so it can find Elasticsearch on the network. 

Once Elasticsearch is up and running, [connect it to Liferay DXP](./connecting-to-elasticsearch.md).

## Related Topics

[Liferay Enterprise Search](../../liferay_enterprise_search.rst) \
[Search Pages](../../search-pages-and-widgets/working-with-search-pages/search-pages.md) \
[Administering and Tuning Search](../../search_administration_and_tuning.rst) \
[Search Configuration Reference Guide](../../search-configuration-reference.md) \
