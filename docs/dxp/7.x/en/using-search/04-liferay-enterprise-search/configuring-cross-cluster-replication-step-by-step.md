# Configuring Cross-Cluster Replication, Step-By-Step

SAME DIAGRAM USED IN INTRO ARTICLE

ALSO, ADD A BEFORE AND AFTER LOOK AT THE CONNECTIONS TAB

## Install the Cross-Cluster Replication Module

<!-- Does a docker container constitute a single node in a cluster? We'll need to instruct to use the les/mp app in control panel to install the ccr module into each dxp node that will use a read-only connection to the follower cluster. -->

1. Download the Liferay DXP Cross-Cluster Replication for Elasticsearch LPKG from the LES downloads page.

1. Install it into the docker image for the local Liferay DXP instance.<!-- how do we do this? -->

## Configure the Elasticsearch Clusters

This example uses two Elasticsearch clusters, each with a copy of the same indexes.

A vanilla Liferay DXP 7.2 installation contains the following indexes:

liferay-0
liferay-20101
liferay-search-tuning-rankings
liferay-search-tuning-synonyms-liferay-20101
workflow-metrics-instances
workflow-metrics-nodes
workflow-metrics-processes
workflow-metrics-sla-instance-results
workflow-metrics-sla-task-results
workflow-metrics-tokens

All the Elasticsearch clusters being used by Liferay DXP (2 clusters in this example) need these indexes.

### Configure Elasticsearch Server 1

In our example setup, the first Elasticsearch cluster is a REMOTE mode cluster with no special configuration required: it accepts reads and writes from it's local Liferay DXP node, and write requests from the Liferay DXP nodes that are in a separate data center.

Configure its `elasticsearch.yml` by specifiying a sensible cluster name:

```json
cluster.name: LiferayElasticsearchCluster_aES
ALTERNATIVE cluster.name: LiferayElasticsearchCluster_REMOTE
```

Start the server. If you're in the root of the server directory, execute

```bash
./bin/elasticssearch
```
<!-- a docker friendly option? -->

If you're just trying this out and don't yet have the proper license, start an Elasticsearch trial:

```bash
curl -XPOST "http://localhost:9200/_license/start_trial?acknowledge=true"
```

> On Elasticsearch 6, use
> 
> `curl -XPOST "http://localhost:9200/_xpack/license/start_trial?acknowledge=true"`

<!-- a docker friendly option? -->

You'll see a `- valid` message in your log when it installs successfully: 

```bash
[2020-02-26T10:19:36,420][INFO ][o.e.l.LicenseService     ] [user-pc] license [lf263a315-8da3-41f7-8622-lfd7cc14cae29] mode [trial] - valid
```

### Configure Elasticsearch Server 2 

The second Elasticsearch cluster is going to contain the follower (read-only) indexes in the second data center, and will be local to a Liferay DXP node.

Configure its `elasticsearch.yml`, specifying a `http.port` and `transport.port` that won't collide with the other Elasticsearch server:

```json
cluster.name: LiferayElasticsearchCluster_bES
http.port: 9202
transport.port: 9500-9600
```

Start the server. If you're in the root of the server directory, execute

```bash
./bin/elasticssearch
```
<!-- a docker friendly option? -->

If you're just trying this out and don't yet have the proper license, start an Elasticsearch trial:

```bash
curl -XPOST "http://localhost:9202/_license/start_trial?acknowledge=true"
```

> On Elasticsearch 6, use
> 
> `curl -XPOST "http://localhost:9200/_xpack/license/start_trial?acknowledge=true"`
<!-- a docker friendly option? -->

## Configure the Liferay DXP Connectors to Elasticsearch

### Configure the Remote Liferay DXP Node

Admin prepares remote Liferay Instance, that talks with the REMOTE mode Elasticsearch server. <!--aDXP, or A.-->

Provide a `portal-ext.properties` file with these contents:

```properties
cluster.link.enabled=true
cluster.link.channel.name.control=test-control-channel
cluster.link.channel.name.transport.0=test-transport-channel
module.framework.properties.osgi.console=11311
```

Then configure the Liferay Connector to Elasticsearch X [6 or 7], by providing a configuration file

```bash
osgi/configs/com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
```

Give it these contents:

```properties
logExceptionsOnly="false"
operationMode="REMOTE"
clusterName = "LiferayElasticsearchCluster_aES"
```

<!-- (Liferay Elasticsearch Configuration aDXP(A): Cluster is aES(X), or ends in _REMOTE.) -->

Start the Liferay DXP server and reindex at Control Panel &rarr; Configuration &rarr; Search. Click _Reindex All_ in the Index Actions tab.

## Replicate the Leader Indexes into the Local Elasticsearch Server

Here you'll use two Elasticsearch API endpoints, `_cluster/settings` and `_ccr/follow`, from the local follower cluster.

You can use Kibana to call Elasticsearch's APIs, but Here we'll show a cURL call that does the same thing: 

First, from the local cluster that will contain the follower indexes, set the leader cluster:

```bash
curl -XPUT "http://localhost:9202/_cluster/settings" -H 'Content-Type: application/json' -d'{  "persistent" : {    "cluster" : {      "remote" : {        "leader" : {          "seeds" : [            "127.0.0.1:9300"           ]        }      }    }  }}'
```

Messages including `updating [cluster.remote.leader.seeds]` will appear in the log:

```bash
[2020-02-26T14:15:54,752][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,758][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,759][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,760][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,815][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,819][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,820][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,821][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
```


Once the leader is configured, use the Follow API to perform the initial index replication of the leader's indexes into the local/follower cluster:

```bash
curl -XPUT "http://localhost:9202/liferay-20101/_ccr/follow?wait_for_active_shards=1" -H 'Content-Type: application/json' -d'{  "remote_cluster" : "leader",  "leader_index" : "'liferay-20101'"}'
```

Messages indicating that the shard has started and that the leader is being tracked will appear in the console:

```bash
[2020-02-26T14:25:06,806][INFO ][o.e.c.r.a.AllocationService] [russell-pc] Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[liferay-20101][0]]]).
[2020-02-26T14:25:06,867][DEBUG][o.e.a.a.c.s.r.RestoreClusterStateListener] [russell-pc] restore of [_latest_/_latest_] completed
[2020-02-26T14:25:07,019][INFO ][o.e.x.c.a.ShardFollowTasksExecutor] [russell-pc] [liferay-20101][0] Starting to track leader shard [liferay-20101][0]
[2020-02-26T14:25:07,080][INFO ][o.e.x.c.a.ShardFollowNodeTask] [russell-pc] [liferay-20101][0] following leader shard [liferay-20101][0], follower global checkpoint=[177], mapping version=[12], settings version=[0], aliases version=[0]
```

Repeat the above curl command for all the indexes you see listed at Control Panel &rarr; Configuration &rarr; Search &rarr; Field Mappings. For example, these indexes are present in most systems:

liferay-0
liferay-20101
liferay-search-tuning-rankings
liferay-search-tuning-synonyms-liferay-20101
workflow-metrics-instances
workflow-metrics-nodes
workflow-metrics-processes
workflow-metrics-sla-instance-results
workflow-metrics-sla-task-results
workflow-metrics-tokens

Now the second Elasticsearch cluster knows how to replicate from the remote leader cluster. The last step is to wire up the final Liferay DXP node so it can read from this second Elasticsearch server's indexes, and write to the first (remote/leader) Elasticsearch cluster.

### Configure the Local Liferay DXP Node

Provide a `portal-ext.properties` file with these contents:

```properties
cluster.link.enabled=true
cluster.link.channel.name.control=test-control-channel
cluster.link.channel.name.transport.0=test-transport-channel
module.framework.properties.osgi.console=11312
```

Then configure the Liferay Connector to Elasticsearch X [6 or 7], by providing a configuration file called

```bash
osgi/configs/com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
```

This configures the write-enabled connection to the remote Elasticsearch cluster with the leader indexes. Give it these contents:

```properties
logExceptionsOnly="false"
operationMode="REMOTE"
clusterName = "LiferayElasticsearchCluster_aES"
```

Now configure the read-only connection to the local Elasticsearch server with the follower indexes. Provide a configuration file named 

```bash
osgi/configs/com.liferay.portal.search.elasticsearch.cross.cluster.replication.internal.configuration.CrossClusterReplicationConfiguration.config
```

Give it these contents:

```properties
ccrEnabled="true"
clusterName="LiferayElasticsearchCluster_bES"
transportAddresses="localhost:9500"
```


<!-- END INSTRUCTIONS -->


