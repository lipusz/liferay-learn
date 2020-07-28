# Getting Started with Elasticsearch

> Latest Supported Elasticsearch Version: 7.7 \
> Available: \
>    Liferay DXP 7.2 SP1+ \
>    Liferay CE/DXP 7.3

<!-- need to iron out the version stuff: https://liferay.slack.com/archives/CNJBTQNQY/p1592942370027900 -->

You should always install the latest [supported version](https://help.liferay.com/hc/sections/360002103292-Compatibility-Matrix) of Elasticsearch for Liferay DXP with the latest Liferay Connector to Elasticsearch. A connector application is installed by default on all Liferay DXP installations, but it's not necessarily the newest connector.

Whether installing DXP and Elasticsearch clusters in a local on-premise configuration, installing single-node testing or development servers for each, or setting up a production Docker installation, this guide walks you through the setup procedure for the latest supported version of Elasticsearch.

To set up an example Liferay DXP cluster with a remote Elasticsearch connection, see the [Clustering for High Availability](../../../installation-and-upgrades/setting-up-liferay-dxp/clustering-for-high-availability/example-creating-a-simple-dxp-cluster.md) documentation.

## Prerequisites

There are some prerequisite details to iron out that aren't specific to the Elasticsearch or Liferay DXP configuration.

### Production-Like Local Environment: Add Hosts

You can skip this if you'll set up a testing environment using `localhost` or Docker containers. For a production-like setup on your local machine, add the hosts for Liferay DXP and the Elasticsearch cluster. Add this to your operating system's `path/to/etc/hosts` file:

```properties
<your IP> es-node1
<your IP> es-node2
<your IP> es-node3
<your IP> dxp.liferay.com
```

Use the real IP address of your system, not the loopback address `127.0.0.1`.

### Docker Containers: Add Hosts

> Skip this step if your system is not using Docker containers.

The Liferay DXP container(s) must recognize the Elasticsearch IP(s) to establish a connection. Add `/etc/hosts/` entries on the DXP nodes that map the Elasticsearch container name to the Elasticsearch server host IP address. This can be established during the `docker run` phase by passing an `--add-host elasticsearch:[IP address]` argument for each Elasticsearch node.

To obtain the IP addresses of all running containers, run 

```bash
docker network inspect bridge
```

The example IP value presented here is `172.17.0.2`:

```bash
docker run -it --name dxp-1 --add-host elasticsearch7:172.17.0.2 ...
```

### Increase the Virtual Memory of Elasticsearch's Host

> Skip this step if your system is not using Docker containers.

You can skip this step if you're setting up a `localhost` testing environment with a downloaded Elasticsearch archive. Elasticsearch requires a higher _mmap count_ (for mapping the directory holding its indexes into memory) than most operating systems are configured for by default. On Linux and as the root user, run

```bash
sysctl -w vm.max_map_count=262144
```

See [Elasticsearch's documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/vm-max-map-count.html) to learn more, including how to set it permanently.

CONCLUDE THIS

## Related Topics

[Liferay Enterprise Search](../../liferay_enterprise_search.rst)
[Search Pages](../../search-pages-and-widgets/working-with-search-pages/search-pages.md)
[Administering and Tuning Search](../../search_administration_and_tuning.rst)
[Search Configuration Reference Guide](../../search-configuration-reference.md)

