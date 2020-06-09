# Exercise: Upgrading the Example Liferay DXP Cluster

> Prerequisite: Configure the simple example cluster in the [Clustering for High Availability](../../../installation-and-upgrades/setting-up-liferay-dxp/clustering_for_high_availability.rst) section.

The example cluster configured [here](../../../installation-and-upgrades/setting-up-liferay-dxp/clustering-for-high-availability/example-creating-a-simple-dxp-cluster.md) uses the  same Elasticsearch version as the default bundled Elasticsearch, to get a DXP cluster up and running quickly with a remote Elasticsearch server. While convenient, this is not ideal. If there's no compelling reason otherwise, use the latest supported version of Elasticsearch. There are a few additional steps to those basic instructions:

1. Start an Elasticsearch 7 Docker container.
1. Blacklist the default connector to Elasticsearch 6 and the bundled APIs.
1. Install a connector for the newest version (includes new API versions).
1. Restart DXP and Reindex into Elasticsearch 7.

To follow these instructions you must have configured the example cluster as per the documentation [here](../../../installation-and-upgrades/setting-up-liferay-dxp/clustering-for-high-availability/example-creating-a-simple-dxp-cluster.md).

## Start an Elasticsearch 7 Instance

Create and configure an Elasticsearch server:

1.Set a local folder for storing an Elasticsearch server's data volume. For example,
 
   ```bash
   mkdir -p elasticsearch/es_data_volume
   ```
 
1. Start an Elasticsearch container named elasticsearch.

   <!-- this doesn't match the recommendations from elastic's docs. see https://www.elastic.co/guide/en/elasticsearch/reference/7.7/docker.html-->
   ```bash
   docker run -it --name elasticsearch -p 9200:9200 -p 9300:9300 -e cluster.name=LiferayElasticsearchCluster -e ES_JAVA_OPTS="-Xms512m -Xmx512m" -v $(pwd)/elasticsearch/es_data_volume:/usr/share/elasticsearch/data elasticsearch:7.7.1
   ```

   <!-- look into this and solve it -->
   If the container reports

   ```bash
   max virtual memory areas vm.max_map_count [xxxxx] is too low, increase to at least [xxxxxx]
   ```

   set `vm.max_map_count` to a sufficient value using a command like this one: 

   ```bash
   sudo sysctl -w vm.max_map_count=[xxxxxx]``. Then start the container.
   ```

1. Install the required Elasticsearch plugins.

   ```bash
   docker exec -it elasticsearch bash -c '/usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-icu && /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-kuromoji && /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-smartcn && /usr/share/elasticsearch/bin/elasticsearch-plugin install analysis-stempel'
   ```

## Blacklisting the Elasticsearch 6 Connector and APIs

Before installing the new connector, blacklist the Elasticsearch 6 connector and APIs.

### Blacklisting Elasticsearch 6 on dxp-1

In the local folder you have bind mounted to the DXP container node called `dxp-1`, run

```bash
cat <<EOT >> dxp-1/files/osgi/configs/com.liferay.portal.bundle.blacklist.internal.BundleBlacklistConfiguration.config
blacklistBundleSymbolicNames=[ \
	"com.liferay.portal.search.elasticsearch6.api", \
	"com.liferay.portal.search.elasticsearch6.impl", \
	"com.liferay.portal.search.elasticsearch6.spi", \
	"com.liferay.portal.search.elasticsearch6.xpack.security.impl", \
	"Liferay Connector to X-Pack Security [Elastic Stack 6.x] - Impl" \
]
EOT
```

### Blacklisting Elasticsearch 6 on dxp-2

In the local folder you have bind mounted to the DXP container node called `dxp-2`, run

```bash
cat <<EOT >> dxp-1/files/osgi/configs/com.liferay.portal.bundle.blacklist.internal.BundleBlacklistConfiguration.config
blacklistBundleSymbolicNames=[ \
	"com.liferay.portal.search.elasticsearch6.api", \
	"com.liferay.portal.search.elasticsearch6.impl", \
	"com.liferay.portal.search.elasticsearch6.spi", \
	"com.liferay.portal.search.elasticsearch6.xpack.security.impl", \
	"Liferay Connector to X-Pack Security [Elastic Stack 6.x] - Impl" \
]
EOT
```

## Installing and Configuring the Liferay DXP Connector to Elasticsearch 7

The configuration file shown in the [example clustering article](../../../installation-and-upgrades/setting-up-liferay-dxp/clustering-for-high-availability/example-creating-a-simple-dxp-cluster.md) is for the Elasticsearch 6 connector that's installed by default on Liferay DXP 7.2 and 7.3. Once you blacklist Elasticsearch 6, it has no effect. 

1. Download the Liferay Connector to Elasticsearch 7:

   - [CE](https://web.liferay.com/en/marketplace/-/mp/application/170642090)
   - [DXP](https://web.liferay.com/en/marketplace/-/mp/application/170390307)

1. [Install the LPKG](../../../system-administration/installing-and-managing-apps/installing-apps/installing-apps.md).
    <!-- should we have them install via the MP app in portal since we're in the docker container? -->

### Configuring Elasticsearch 7 for dxp-1

In the local folder you bind mounted to the DXP container node `dxp-1` run

```bash
cat <<EOT >> dxp-1/files/osgi/configs/com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
operationMode="REMOTE"
transportAddresses="elasticsearch:9300"
clusterName="LiferayElasticsearchCluster"
EOT
```

### Configuring Elasticsearch 7 for dxp-2

In the local folder you bind mounted to the DXP container node `dxp-2` run

```bash
cat <<EOT >> dxp-1/files/osgi/configs/com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
operationMode="REMOTE"
transportAddresses="elasticsearch:9300"
clusterName="LiferayElasticsearchCluster"
EOT
```

Restart both Liferay DXP nodes and reindex your search and spell check indexes. Both reindex actions are carried out at Control Panel &rarr; Configuration &rarr; Search, in the Index Actions tab.
