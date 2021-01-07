# Configuring an Example CCR Installation: Replicating Between Data Centers

> **Liferay Enterprise Search (LES) Subscribers**

Configure Liferay DXP's Cross-Cluster Replication module and Elasticsearch to set up a read-write connection from one Elasticsearch cluster to one Liferay DXP cluster node, and a read connection from another Elasticsearch cluster to a second Liferay DXP cluster node:

![With Cross-Cluster Replication, disparate data centers can hold synchronized Elasticsearch clusters with Liferay DXP indexes.](./configuring-an-example-ccr-installation-replicating-between-data-centers/images/01.png)

This example uses two single-node Elasticsearch clusters on `localhost`, each with a copy of the same indexes. This represents the simplest scenario you can configure to reap the data locality and disaster recovery benefits of Cross-Cluster Replication.

A vanilla Liferay DXP installation contains the indexes presented in the introductory [Cross Cluster Replication](./cross-cluster-replication.md#liferay-dxp-decide-which-indexes-to-replicate-from-the-remote-cluster) article. All the Elasticsearch clusters being used by Liferay DXP (2 clusters in this example) need these indexes.

Any Elasticsearch API calls provided here are in a format that allows you to copy and paste them directly into Kibana's Dev Tools console, which can be accessed via a separate Kibana installation or through the [LES Monitoring widget](../monitoring-elasticsearch.md).

```note::
   To use Kibana, remember that you have multiple Elasticsearch clusters (two single-node clusters in this example) running. The ``elasticsearch.hosts: [ "http://localhost:<port>" ]`` setting in Kibana's ``kibana.yml`` file must point to the correct port when managing the indexes and other configurations described below to avoid mixing the leader and the follower clusters. In this article, we assume that your leader Elasticserach cluster node is configured to use ``9200`` while the follower node is using ``9202`` as HTTP port.
```

## Cluster Liferay DXP 

For the example here, each Liferay DXP node requires a `Liferay Home/portal-ext.properties` file with the following contents:

```properties
cluster.link.enabled=true
```

This is a simplistic clustering configuration. See the [documentation on clustering](../../../installation-and-upgrades/setting-up-liferay-dxp/clustering-for-high-availability/clustering-for-high-availability.md) for information on how clustering works with Liferay DXP.

## Install Required Elasticsearch Plugins

Make sure you install the Elasticsearch [plugins Liferay DXP needs](../../installing-and-upgrading-a-search-engine/elasticsearch/installing-elasticsearch.md#install-elasticsearch):

- `analysis-icu`
- `analysis-stempel`
- `analysis-kuromoji`
- `analysis-smartcn`

## Prerequisite for Security: Configure Authentication and Encryption

To encrypt communication (TLS/SSL) and enable user authentication between the Liferay DXP and the Elasticsearch nodes, you must [configure security](../../installing-and-upgrading-a-search-engine/elasticsearch/securing-elasticsearch.md).

The high level steps are,

1. Configure X-Pack Security in your Elasticsearch clusters and make sure the node certificates are signed by the same CA and the security settings of the Leader and the Follower clusters match.

   ```note::
      TLS/SSL must be enabled for the HTTP and Transport layers on the Follower Elasticsearch cluster nodes. Liferay DXP connects to the Follower cluster over HTTP to re-follow the company indexes after a full reindex is performed.
   ```

1. Configure the DXP nodes. Configure the production mode settings for Elasticsearch in the `ElasticsearchConfiguration.config` file,  then configure the connections separately in `ElasticsearchConnectionConfiguration-[ccr/remote].config` files. Include the security settings on each connection. See [Securing Elasticsearch](../../installing-and-upgrading-a-search-engine/elasticsearch/securing-elasticsearch.md) for details. The [Configuring CCR in a Local Follower Data Center](./configuring-ccr-in-a-local-follower-data-center.md) article covers configuring security for the read-only CCR connection from the local DXP nodes.

The example configurations are provided in full [here](./ccr-basic-use-case-config-reference.md).

## Install the Cross-Cluster Replication Module

1. Download the Liferay DXP Cross-Cluster Replication for Elasticsearch LPKG from the [LES downloads page](https://customer.liferay.com/downloads).

1. [Install the LPKG](../../system-administration/installing-and-managing-apps/installing-apps/installing-apps.md) into all DXP nodes.

If all your prerequisite tasks are completed and the Cross-Cluster Replication module is installed, continue by [configuring the servers in your remote data center](./configuring-ccr-in-a-remote-leader-data-center.md).
