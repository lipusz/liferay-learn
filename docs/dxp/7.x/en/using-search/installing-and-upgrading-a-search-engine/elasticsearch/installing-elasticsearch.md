# Installing Elasticsearch

> Latest Supported Elasticsearch Version: 7.7

You should always install the latest supported version of Elasticsearch for Liferay DXP.

## Obtain and Configure Elasticsearch

1. Obtain Elasticsearch. See [Elastic's](https://www.elastic.co) website for a local download or [pull a docker image](https://www.docker.elastic.co/):

1. If you downloaded Elasticsearch directly, install it by extracting its archive to the system where you want it to run.

1. Install some required Elasticsearch plugins. Run these commands from the `bin` folder:

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

   This is demonstrated in the example cluster configured [here](../../../installation-and-upgrades/setting-up-liferay-dxp/clustering-for-high-availability/example-creating-a-simple-dxp-cluster.md).

1. Name your Elasticsearch cluster. Each node's `config/elasticearch.yml` must have the same cluster name:

   ```yaml
   cluster.name: LiferayElasticsearchCluster
   ```

   > Docker: For a simple single-node test cluster, you can specify the cluster name (and any other configuration options) when first creating and starting the container (see below for example).
   > 
   > For anything more than one node or requiring more extensive configuration, it's best to use a [docker-compose.yml file](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/docker.html#docker-compose-file).


   Name each node in the cluster:

   ```yaml
   node.name: liferay-es-node-1
   ```

1. Start the Elasticsearch cluster:

   If using the un-packaged Elasticsearch archive, from the `bin` folder, run 

   ```bash
   ./elasticsearch
   ```

   If using a Docker container to drive a multi-node Elasticsearch cluster configured with a `docker-compose.yml` file, run

   ```bash
   docker-compose up
   ```

   A single-node test cluster can be started and configured at the same time with a `docker run` command:


   ```bash
   docker run -p 9200:9200 -p 9300:9300 -e "cluster.name=LiferayElasticsearchCluster" -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:7.7.1
   ```

Elasticsearch starts, and one of its status messages includes a transport address: 

```sh
[2019-04-01T16:55:50,127][INFO ][o.e.t.TransportService   ] [HfkqdKv] publish_address {127.0.0.1:9300}, bound_addresses {[::1]:9300}, {127.0.0.1:9300}
```

Take note of this address; you'll need to give it to your DXP server(s) so it can find Elasticsearch on the network. 

## Configure the Liferay DXP to Elasticsearch Connections 

Stop each Liferay DXP server node before completing these steps.

1. Download the Liferay Connector to Elasticsearch 7:

   - [CE](https://web.liferay.com/en/marketplace/-/mp/application/170642090)
   - [DXP](https://web.liferay.com/en/marketplace/-/mp/application/170390307)

1. [Install the LPKG](../../../system-administration/installing-and-managing-apps/installing-apps/installing-apps.md) by placing it in the folder

   ```bash
   [Liferay Home]/deploy
   ```

   When you start the server (not yet) the LPKG is processed and deployed.

1. Create a configuration file for the Elasticsearch connector: 

   ```bash
   touch [Liferay Home]/osgi/configs/com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
   ```

   A one-node cluster with the most basic configuration, suitable for testing, might have these contents:

   ```properties
   # look for an ES server other than the embedded one
   operationMode="REMOTE"
   # as printed in the Elasticsearch log
   transportAddresses="localhost:9300"
   # cluster name as defined in elasticsearch.yml
   clusterName="LiferayElasticsearchCluster" 
   ```

   If using a docker container with a local bind mount for each node, make sure you put this file there.

1. Restart Liferay DXP and reindex your search and spell check indexes. Both reindex actions are carried out at Control Panel &rarr; Configuration &rarr; Search, in the Index Actions tab.

When the installation is completed, that's when the configuring and searching can begin.

## Related Topics

[Liferay Enterprise Search](../../liferay_enterprise_search.rst)
[Search Pages](../../search-pages-and-widgets/working-with-search-pages/search-pages.md)
[Administering and Tuning Search](../../search_administration_and_tuning.rst)
[Search Configuration Reference Guide](../../search-configuration-reference.md)

