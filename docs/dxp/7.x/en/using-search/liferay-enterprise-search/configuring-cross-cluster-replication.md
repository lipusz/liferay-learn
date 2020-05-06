# Configuring Cross-Cluster Replication

> **Liferay Enterprise Search (LES) Subscribers**

<!-- SAME DIAGRAM USED IN INTRO ARTICLE

ALSO, ADD A BEFORE AND AFTER LOOK AT THE CONNECTIONS TAB
-->
The commands that involve calling Elasticsearch APIs are provided in a format that allows you to  copy and paste them directly into [Kibana's Dev Tools](./monitoring-elasticsearch.md) console.

<!-- From Tibor: Highlight that the guide is super-simplified and deals with setting up a 1-1 node ES clusters (leader and follower) running on localhost. A prod-ready environment needs different settings.-->
<!-- From Russ: We should just adapt to those settings instead of saying "this guide shows steps that you can't follow for a real setup." I think we need to elevate our docs game for CCR. -->

## Prerequisite for Elasticsearch 6: Enable Soft Deletes

[Soft deletes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/ccr-requirements.html) must be enabled for all existing indexes. This is not done by default on Elasticsearch 6. Before proceeding, read [here](./configuring-ccr-enabling-soft-deletes-on-elasticsearch-6.md) to configure soft deletes on your Elasticsearch 6 indexes, then resume reading here to set up CCR.

## Install the Cross-Cluster Replication Module

1. Download the Liferay DXP Cross-Cluster Replication for Elasticsearch LPKG from the LES downloads page.

1. [Install the LPKG](../../system-administration/installing-and-managing-apps/installing-apps/installing-apps.md) into the local Liferay DXP instance.

## Configure the Elasticsearch Clusters

This example uses two single-node Elasticsearch clusters on `localhost`, each with a copy of the same indexes.

A vanilla Liferay DXP 7.2 installation contains the indexes presented in the [Cross Cluster Replication](./cross-cluster-replication.md#liferay-dxp-decide-whichindexes-to-replicate-from-the-remote-cluster) article.

All the Elasticsearch clusters being used by Liferay DXP (2 clusters in this example) need these indexes.

### Configure the Leader Elasticsearch Cluster

In our example setup, the first Elasticsearch cluster is a REMOTE mode cluster with no special configuration required: it accepts reads and writes from it's local Liferay DXP node, and write requests from the Liferay DXP nodes that are in a separate data center.

Configure its `elasticsearch.yml` by specifying a sensible cluster name:

```json
cluster.name: LiferayElasticsearchCluster_LEADER
```
Start the server. If you're in the root of the server directory, execute

```bash
./bin/elasticssearch
```

If you're just trying this out and don't yet have the proper license, start an [Elasticsearch trial](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/start-trial.html):

```json
POST /_license/start_trial?acknowledge=true
```

> On Elasticsearch 6, use
> 
> `POST _xpack/license/start_trial?acknowledge=true`

You'll see a `- valid` message in your log when it installs successfully: 

```bash
[2020-02-26T10:19:36,420][INFO ][o.e.l.LicenseService     ] [user-pc] license [lf263a315-8da3-41f7-8622-lfd7cc14cae29] mode [trial] - valid
```

### Configure the Follower Elasticsearch Cluster 

The second Elasticsearch cluster is going to contain the follower (read-only) indexes in the second data center, and will be local to a Liferay DXP node.

Configure its `elasticsearch.yml`, specifying a `http.port` and `transport.port` that won't collide with the other Elasticsearch server:

```json
cluster.name: LiferayElasticsearchCluster_FOLLOWER
http.port: 9202
transport.port: 9500-9600
```

Start the server. If you're in the root of the server directory, execute

```bash
./bin/elasticssearch
```

If you're just trying this out and don't yet have the proper license, start an Elasticsearch trial:

```json
POST /_license/start_trial?acknowledge=true
```

> On Elasticsearch 6, use
> 
> `POST _xpack/license/start_trial?acknowledge=true`

## Configure the Liferay DXP Connectors to Elasticsearch

### Configure the Remote Liferay DXP Node

Admin prepares remote Liferay Instance, that talks with the REMOTE mode Elasticsearch server.

Provide a `portal-ext.properties` file with these contents:

```properties
cluster.link.enabled=true
cluster.link.channel.name.control=test-control-channel
cluster.link.channel.name.transport.0=test-transport-channel
module.framework.properties.osgi.console=11311
```

Then configure the Liferay Connector to Elasticsearch X [6 or 7], by providing a configuration file in the `Liferay Home/osgi/configs` folder. If using Elasticsearch 7, name it

```bash
com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
```

If using Elasticsearch 6, the configuration file is named

```bash
com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config
```

Give it these contents:

```properties
operationMode="REMOTE"
clusterName = "LiferayElasticsearchCluster_LEADER"
```

```tip::
   During development and testing, it's useful to set ``logExceptionsOnly="false"`` in the configuration file as well.
```

Start the Liferay DXP server.

## Replicate the Leader Indexes into the Local Elasticsearch Server

Here you'll use two Elasticsearch API endpoints, `_cluster/settings` and `_ccr/follow`, from the local follower cluster.

You can use [Kibana](./monitoring-elasticsearch.md) to call Elasticsearch's APIs. 

First, from the local cluster that will contain the follower indexes, set the leader cluster:

```json
PUT /_cluster/settings
{
  "persistent" : {
    "cluster" : {
      "remote" : {
        "leader" : {
          "seeds" : [
            "127.0.0.1:9300" 
          ]
        }
      }
    }
  }
}
```

Messages including `updating [cluster.remote.leader.seeds]` will appear in the log:

```bash
[2020-02-26T14:15:54,752][INFO ][o.e.c.s.ClusterSettings  ] [user-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,758][INFO ][o.e.c.s.ClusterSettings  ] [user-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,759][INFO ][o.e.c.s.ClusterSettings  ] [user-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,760][INFO ][o.e.c.s.ClusterSettings  ] [user-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,815][INFO ][o.e.c.s.ClusterSettings  ] [user-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,819][INFO ][o.e.c.s.ClusterSettings  ] [user-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,820][INFO ][o.e.c.s.ClusterSettings  ] [user-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,821][INFO ][o.e.c.s.ClusterSettings  ] [user-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
```


Once the leader is configured, use the Follow API to perform the initial index replication of the leader's indexes into the local/follower cluster:

```json
PUT /liferay-20101/_ccr/follow?wait_for_active_shards=1
{
  "remote_cluster" : "leader",
  "leader_index" : "liferay-20101"
}
```

```note::
   The value ``leader`` is used in the API calls above, as it is the default `alias to the remote cluster <https://www.elastic.co/guide/en/elasticsearch/reference/7.x/ccr-getting-started.html#ccr-getting-started-remote-cluster>`_: if you use a different alias, change this value in the API calls, and set the same value in the ``remoteClusterAlias`` property of the ``CrossClusterReplicationConfiguration``.
```

Messages indicating that the shard has started and that the leader is being tracked will appear in the console:

```bash
[2020-02-26T14:25:06,806][INFO ][o.e.c.r.a.AllocationService] [user-pc] Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[liferay-20101][0]]]).
[2020-02-26T14:25:06,867][DEBUG][o.e.a.a.c.s.r.RestoreClusterStateListener] [user-pc] restore of [_latest_/_latest_] completed
[2020-02-26T14:25:07,019][INFO ][o.e.x.c.a.ShardFollowTasksExecutor] [user-pc] [liferay-20101][0] Starting to track leader shard [liferay-20101][0]
[2020-02-26T14:25:07,080][INFO ][o.e.x.c.a.ShardFollowNodeTask] [user-pc] [liferay-20101][0] following leader shard [liferay-20101][0], follower global checkpoint=[177], mapping version=[12], settings version=[0], aliases version=[0]
```

Repeat the above PUT call for all the indexes you see listed at Control Panel &rarr; Configuration &rarr; Search &rarr; Field Mappings. For example, these indexes are present in most systems:

- liferay-0
- liferay-20101
- liferay-search-tuning-rankings
- liferay-search-tuning-synonyms-liferay-20101
- workflow-metrics-instances
- workflow-metrics-nodes
- workflow-metrics-processes
- workflow-metrics-sla-instance-results
- workflow-metrics-sla-task-results
- workflow-metrics-tokens

Now the second Elasticsearch cluster knows how to replicate from the remote leader cluster. The last step is to wire up the final Liferay DXP node so it can read from this second Elasticsearch server's indexes, and write to the first (remote/leader) Elasticsearch cluster.

### Configure the Local Liferay DXP Node

Provide a `portal-ext.properties` file with these contents:

```properties
cluster.link.enabled=true
cluster.link.channel.name.control=test-control-channel
cluster.link.channel.name.transport.0=test-transport-channel
module.framework.properties.osgi.console=11312
```

Then configure the Liferay Connector to Elasticsearch X [6 or 7], by providing a configuration file in the `Liferay Home/osgi/configs` folder. If using Elasticsearch 7, name it

```bash
osgi/configs/com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
```

If using Elasticsearch 6, the configuration file is named

```bash
osgi/configs/com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config
```

This file configures the write-enabled connection to the remote Elasticsearch cluster with the leader indexes. Give it these contents:

```properties
logExceptionsOnly="false"
operationMode="REMOTE"
clusterName = "LiferayElasticsearchCluster_LEADER"
```

Now configure the read-only connection to the local Elasticsearch server with the follower indexes. Provide a configuration file named 

```bash
osgi/configs/com.liferay.portal.search.elasticsearch.cross.cluster.replication.internal.configuration.CrossClusterReplicationConfiguration.config
```

Give it these contents:

```properties
ccrEnabled="true"
clusterName="LiferayElasticsearchCluster_FOLLOWER"
transportAddresses="localhost:9500"
```

<!-- From Tibor: Add note that the actual port number may be different depending on in which order you started the Leader and the Follower clusters if both are running on localhost.-->
<!-- From Russ: I can do this, but I'm not convinced this is possible with these instructions. We set the transport port range to 9500-9600 for this ES cluster, and we left the other with the default setting (9300-9400), so will the startup order matter?-->
