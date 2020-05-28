# Configuring CCR: A Basic Use Case

> **Liferay Enterprise Search (LES) Subscribers**

Configure Liferay DXP's Cross-Cluster Replication module and Elasticsearch to set up a read-write connection from one Elasticsearch server to one Liferay DXP cluster node, and a read connection from another Elasticsearch server to a second Liferay DXP cluster node:

```important::
   To use CCR, all of your DXP cluster nodes must be running Liferay DXP 7.2 Fix Pack 5+ / Service Pack 2+.
```

![With Cross-Cluster Replication, disparate data centers can hold synchronized Elasticsearch clusters with Liferay DXP indexes.](./configuring-ccr-a-basic-use-case/images/01.png)

This example uses two single-node Elasticsearch clusters on `localhost`, each with a copy of the same indexes. This represents the simplest scenario you can configure to reap the data locality and disaster recovery benefits of Cross-Cluster Replication.

A vanilla Liferay DXP 7.2 installation contains the indexes presented in the introductory [Cross Cluster Replication](./cross-cluster-replication.md#liferay-dxp-decide-whichindexes-to-replicate-from-the-remote-cluster) article. All the Elasticsearch clusters being used by Liferay DXP (2 clusters in this example) need these indexes.

The Elasticsearch API calls are provided in a format that allows you to  copy and paste them directly into Kibana's Dev Tools console, which can be accessed via a separate Kibana installation or through the [X-Pack Monitoring widget](./monitoring-elasticsearch.md).

```note::
   To use Kibana, remember that you have multiple Elasticsearch clusters (two single-node clusters in this example) running. The `elasticsearch.hosts: [ "http://localhost:<port>" ]` setting in Kibana's `kibana.yml` file must point to the correct port when managing the indexes and other configurations described below to avoid mixing the leader and the follower clusters. In this article, we assume that your leader Elasticserach cluster node is configured to use `9200` while the follower node is using `9201` as HTTP port.
```

<!-- From Tibor: Highlight that the guide is super-simplified and deals with setting up a 1-1 node ES clusters (leader and follower) running on localhost. A prod-ready environment needs different settings.-->
<!-- From Russ: We should just adapt to those settings instead of saying "this guide shows steps that you can't follow for a real setup." I think we need to elevate our docs game for CCR. -->

## Prerequisite for Elasticsearch 6: Enable Soft Deletes

[Soft deletes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/ccr-requirements.html) must be enabled for all existing indexes. This is not done by default on Elasticsearch 6. Before proceeding, read [here](./configuring-ccr-enabling-soft-deletes-on-elasticsearch-6.md) to configure soft deletes on your Elasticsearch 6 indexes, then resume reading here to set up CCR.

If Elasticsearch 6 is not a hard requirement for your system, you should upgrade to Elasticsearch 7 before configuring CCR. 

## Prerequisite for Elasticsearch 7: Install the Liferay Connector to Elasticsearch 7

If you are using Elasticsearch 7, you have to install the [Elasticsearch 7 connector](https://web.liferay.com/marketplace/-/mp/application/170390307) from Marketplace **version `3.0.1+`** (requires DXP 7.2 FP5+/SP2+) on all DXP cluster nodes. Read [Upgrading to Elasticsearch 7](https://help.liferay.com/hc/en-us/articles/360035444872-Upgrading-to-Elasticsearch-7) for details.

## Prerequisite for Security: Configure X-Pack Security

To encrypt the communications (TLS/SSL) and enable user authentication between the DXP and the Elasticsearch nodes, you also have to go through the [Installing Liferay Enterprise Search Security](https://help.liferay.com/hc/en-us/articles/360028711172-Installing-Liferay-Enterprise-Search-Security) article. 

The high level steps are,

1. Configure X-Pack Security in your Elasticsearch clusters and make sure the node certificates are signed by the same CA and the security settings of the Leader and the Follower clusters match.

   ```note::
      TLS/SSL must be enabled on the HTTP layer as well (besides the Transport layer) on the Follower Elasticsearch cluster nodes. It's because DXP connects to the Follower cluster over HTTP to re-follow the company indexes after a full reindex is performed.

2. Configure X-Pack Security on the DXP nodes through the connector configurations. The [Configuring CCR in a Local Follower Data Center](./configuring-ccr-in-a-local-follower-data-center.md) article will show you where you need to add the securtiy related settings for the CCR ("read-only") connection on the local/follower DXP node.

3. If your Elastic Stack is 6.x, you will need to deploy Liferay's [X-Pack Security Connector](https://web.liferay.com/marketplace/-/mp/application/106163963) on all DXP nodes.

## Install the Cross-Cluster Replication Module

1. Download the Liferay DXP Cross-Cluster Replication for Elasticsearch LPKG from the [LES downloads page](https://customer.liferay.com/downloads).

1. [Install the LPKG](../../system-administration/installing-and-managing-apps/installing-apps/installing-apps.md) into the local Liferay DXP instance.

If your prerequisites are completed and the Cross-Cluster Replication module is installed, continue by [configuring the servers in your remote data center](./configuring-ccr-in-a-remote-leader-data-center.md).
