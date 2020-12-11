# Cross-Cluster Replication

> **Liferay Enterprise Search (LES) Subscribers**

In a classic Liferay DXP/search engine installation, one Liferay DXP cluster talks to one Elasticsearch cluster, sending all of its read (e.g., execute a search query) and write (e.g., create a document) requests through one connection to the search engine cluster. In this setup, all search engine servers are located in the same data center (though they can be in a different data center than the Liferay DXP servers).

Addressing concerns about data locality and disaster recovery, Elasticsearch released a [Cross-Cluster Replication (CCR)](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-ccr.html) feature that [LES subscribers](https://www.liferay.com/products/dxp/enterprise-search) can use with Liferay DXP, for Elasticsearch 6.7+ and 7+ (refer to the [LES compatibility matrix](https://www.liferay.com/compatibility-matrix/liferay-enterprise-search) for version compatibility details). With the LES CCR module you can achieve a limited form of multi-data-center deployment, in that it does not allow distributing an Elasticsearch cluster's nodes over multiple data centers, but does allow configuring and connecting separate Elasticsearch clusters in each data center. In this setup, there is at least one cluster holding _leader_ indexes and one cluster holding _follower_ indexes that are replicated from the leader. Follower indexes are only ever used by Liferay DXP for reading data. Leader indexes are always used for writing, but are more flexible in that you can also use them for reading, as in the example below:

![With Cross-Cluster Replication, disparate data centers can hold synchronized Elasticsearch clusters with Liferay DXP indexes.](./cross-cluster-replication/images/01.png)

The diagram above shows a minimal example of CCR. Each data center holds one or more DXP nodes and one Elasticsearch cluster. The US data center holds the leader indexes, where all the DXP nodes write to. The Hungary data center holds the follower indexes that the local (Hungary) DXP nodes read from. Data is replicated one way, from leader to follower.

Liferay DXP has long supported the idea of a distributed cluster, with nodes in disparate locations, via Wide Area Network (WAN) protocols. Liferay DXP's flexibility and Elasticsearch's support for Cross-Cluster Replication can support different system designs.

To set up Cross-Cluster Replication you must

- [Purchase a LES subscription](https://www.liferay.com/products/dxp/enterprise-search)
- Install the CCR Module on all Liferay DXP nodes in the local/follower data center
- Decide which indexes to replicate from the leader cluster
- Configure the Elasticsearch Clusters
- Configure the Liferay DXP Cluster
- Define the Leader and Follower
- Replicate the Leader Indexes

Once youe fully understand the steps, [see a basic, specific use case](./configuring-an-example-ccr-installation-replicating-between-data-centers.md) and get started setting up a local example.

## Liferay DXP: Install the LES Cross-Cluster Replication Module

Any Liferay DXP node that will read from a local cluster's follower indexes and write through a separate connection to the remote cluster's leader indexes must have the CCR module installed. This module is available to download (as a LPKG file) with your LES subscription.

If configuring Liferay DXP cluster nodes to read and write from the same Elasticsearch cluster, configure a normal REMOTE mode connection to the Leader Elasticsearch cluster. The CCR module is unnecessary for these nodes.

##  Liferay DXP: Decide Which Indexes to Replicate from the Remote Cluster

The default Liferay DXP 7.3 indexes in your installation approximate the list below (subject to change):

| Index ID                                      | Index Type    | Index Purpose |
| --------------------------------------------- | ------------- | ------------- |
| liferay-0                                     | System Index  | Searching in the System Settings application |
| liferay-20101                                 | Company Index | Searching the indexed assets of the Liferay DXP Virtual Instance |
| liferay-20101-search-tuning-rankings          | App Index     | Primary data storage for the Result Rankings application (per virtual instance as of Liferay DXP 7.3) |
| liferay-20101-search-tuning-synonyms-liferay  | App Index     | Primary data storage for the Synonym Sets application for the given virtual instance |
| liferay-20101-workflow-metrics-instances              | App Index     | Store data about Workflow Instances for the Workflow Metrics application |
| liferay-20101-workflow-metrics-nodes                  | App Index     | Store data about Workflow Nodes for the Workflow Metrics application |
| liferay-20101-workflow-metrics-processes              | App Index     | Store data about Workflow Processes for the Workflow Metrics application |
| liferay-20101-workflow-metrics-sla-instance-results   | App Index     | Primary storage for SLA results per Workflow Instance for the Workflow Metrics application |
| liferay-20101-workflow-metrics-sla-task-results       | App Index     | Primary storage for SLA results per Workflow Task for the Workflow Metrics application |
| liferay-20101-workflow-metrics-tokens                 | App Index     | Store data about Workflow Tokens for the Workflow Metrics application |
| liferay-20101-your-custom-index                 | Custom Index  | Your own storage and searching needs |

```note::
   Liferay DXP provides APIs for creating and using (writing to and reading from) Elasticsearch indexes that remain completely under your control. See the `Developer Guide <../../developer_guide.rst>`__ for information on using these APIs.
```

Unless your setup reveals a very compelling reason not to, you should replicate all of the Liferay DXP indexes and all of your custom indexes into the follower Elasticsearch cluster. 

## Configure the Elasticsearch Clusters

Set up the Elasticsearch clusters, using versions supported with Liferay DXP that also support Cross-Cluster Replications. See the [LES compatibility matrix](https://help.liferay.com/hc/en-us/articles/360016511651-Liferay-Enterprise-Search-Compatibility-Matrix) for details.

Make sure you Install the Elasticsearch [plugins Liferay DXP needs](https://help.liferay.com/hc/en-us/articles/360028711132-Installing-Elasticsearch#step-three-install-elasticsearch-plugins) and [provide cluster names](https://help.liferay.com/hc/en-us/articles/360028711132-Installing-Elasticsearch#step-four-name-your-elasticsearch-cluster) to differentiate your follower and leader clusters.

CCR requires an Elasticsearch Platinum level license, but [LES customers](./introduction-to-les.md) already have this. If you're testing locally, start a [trial license](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/start-trial.html) on each cluster.

## Configure the Liferay DXP Cluster

Configure the Liferay Clustering behavior first. In the example provided in the tutorial, some configuration will be provided for testing purposes. See the [clustering documentation](../../../installation-and-upgrades/setting-up-liferay-dxp/clustering-for-high-availability/clustering-for-high-availability.md) for more information on setting up a production cluster.

### Configure the DXP Nodes that Reside with the Leader Indexes

Configure the DXP nodes located in the same data center with the remote Elasticsearch cluster containing the leader indexes. Provide the proper Elasticsearch Configuration values (via a `.config` file), then start the DXP nodes that will write to the leader indexes, to make sure they're reading and writing properly.

### Configure the DXP Nodes that Reside with the Follower Indexes

Any DXP nodes that are going to read from local follower Elasticsearch indexes and write to remote leader indexes must have the CCR module deployed and configured (use a `.config` file).

Start the nodes and install the LES app.

## Make the Running Elasticsearch Clusters Know Their Respective Roles

From the local cluster containing the follower indexes, declare the cluster that contains the leader indexes:

- Call the `/_cluster/settings` [Elasticsearch API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-update-settings.html) to define the cluster that acts as the remote cluster containing leader indexes.

<!-- Not after LPS-109204 -->
## Replicate the Leader Indexes

Perform the initial index replication from the leader into the follower: 

- Call the `[indexName]/_ccr/follow` [Elasticsearch API](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/ccr-put-follow.html) to configure the follower indexes on the appropriate cluster.

Ready to get a local example up and running? Proceed to the [Configuring CCR: A Basic Use Case article](./configuring-an-example-ccr-installation-replicating-between-data-centers.md).

