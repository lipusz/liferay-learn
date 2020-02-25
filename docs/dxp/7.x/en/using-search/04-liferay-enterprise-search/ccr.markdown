# Configuring Cross-Cluster Index Replication

In a classic Liferay DXP/Search Engine installation, one Liferay DXP cluster talks to one Elasticsearch cluster, sending all of its read (e.g., execute a search query) and write (e.g., create a document) requests through one connection to the Elasticsearch cluster. All Elasticsearch servers were located in the same data center. Responding to concerns about data locality and disaster recovery, Elasticsearch released a [Cross-Cluster Replication (CCR)](https://www.elastic.co/guide/en/elasticsearch/reference/current/xpack-ccr.html) feature that can be used with Elasticsearch 6.7+. With CCR you can achieve a limited form of multi-data-center deployment, in that it does not allow distributing an Elasticsearch cluster's nodes over multiple data centers, but does allow configuring a separate search engine cluster whose indexes are replicated from a cluster defined as the _leader_.

Liferay DXP has long supported the idea of a distributed cluster, with nodes in disparate locations, via Wide Area Network protocols. Using Liferay's clustering ability with Elasticsearch's Cross-Cluster Replication can look different depending on your needs:

- A Production data center holding the leader and some DXP nodes, and a secondary data center holding the follower and some DXP nodes.

DIAGRAM 1 (the setup we'll show here

- Multiple Data centers, chained replication or with all separately replicating directly from the leader. A chained replication scheme involving more than two data centers. Each data center may or may not have and DXP nodes.

DIAGRAM 2

- Bi-Directional Replication, in which each data center holds a leader index and a follower index. All writes happen to the remote and all reads form the local. How do they sync? Would DXP nodes from each cluster need to write simultaneously to make sure the leaders are in sync?

DIAGRAM 3

No matter what setup you decide on, you must:

##  Decide whether all indexes in the Elasticsearch cluster should be available for reading from Liferay DXP

If you have another system indexing data into the same Elasticsearch cluster you're setting up for Liferay, and you do not wish for those indexes to be replicated to the cluster holding follower indexes, blacklist them before you  enable Cross-Cluster Replication.

By default, all indexes present in the Elasticsearch cluster will be replicated. In most cases, this is the desirable situation.

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

## Configure the Elasticsearch Clusters

Set up the Elasticsearch clusters, using versions supported with Liferay DXP. See the [LES compatibility matrix](https://help.liferay.com/hc/en-us/articles/360016511651-Liferay-Enterprise-Search-Compatibility-Matrix?flash_digest=189c5acefca397bdd5f06a6e0f0d2059ef4fa368) for details.

Make sure you Install the Elasticsearch [plugins Liferay DXP needs](https://help.liferay.com/hc/en-us/articles/360028711132-Installing-Elasticsearch#step-three-install-elasticsearch-plugins), and provide cluster names to differentiate your follower and leader cluster.

CCR requires an Elasticsearch Platinum level license, but LES customers already have this. If you're testing locally, you'll need to install a trial license on each cluster.

## Configure the Liferay DXP Cluster 

Configure the Liferay Clustering behavior first. In the example, some configuration will be provided for testing purposes. See the [clustering documentation](LINK) for more information on setting up a production cluster.

### Configure the Nodes that Reside with the Leader Elasticsearch

Configure the DXP nodes that are located in the same data center with the Elasticsearch cluster containing the leader indexes. Provide the proper Elasticsearch Configuration values.
Fire up the DXP nodes that will write to the remote leader Elasticsearch indexes, make sure they're reading and writing properly.

### Configure the Nodes that Reside with the Follower Elasticsearch

Any DXP nodes that are going to read from a local Follower Elasticsearch  index need to have the CCR module configured. The only configuration in the Elasticsearch [Version] configuration to make sure you enter is ??? anything in the es config?

Before starting the DXP nodes in the same data center as the follower Elasticsearch, provide the CCR module confiuguration. This way, once you install the LPKG from Elasticsearch, your configuration will be read and immediately take effect.

Start the nodes and install the LES app.

## Make the Running Elasticsearch Clusters Know Their Respective Roles

Configure the Leader indexes on the appropriate cluster.

Configure the Follower indexes on the appropriate cluster.

Never reindex from the DXP nodes that are reading from the Follower!

The simplest system using CCR will contain four servers:

1. Liferay DXP A uses a classic (read/write) remote connection to an Elasticsearch server or cluster.
1. Liferay DXP B uses a CCR (read-only) connection to a geographically close Elasticsearch server, Elasticsearch Y, the _follower_, and a write connection to Elasticsearch X, the _leader_.
    Liferay DXP A and Liferay DXP B form a cluster.
1. Elasticsearch X is configured as a remote server for Liferay DXP A, and the _leader_ for Elasticsearch Y.
1. Elasticsearch Y is a follower of Elasticsearch X, replicating its indexes.
    Elasticsearch X and Elasticsearch Y are separate servers, not nodes in a cluster.

Replication is configured at the index level, not at the server/cluster level.


One "Normal" System: Liferay Instance A Reads from and Writes to Elasticsearch Cluster X, whose liferay-[companyId] is the Leader index
One "CCR" Cluster/Index: Liferay Instance B writes to ES X, but Reads from the Follower Cluster/Index, ES Y.Hi, my name is :liferay-waffle:  DXP A. I can read data from and write data to my configured Elasticsearch cluster (:elasticsearch: ES X) just like a normal, pre-CCR system.Hi, my name is :elasticsearch:  ES X. I can do almost everything. :liferay-waffle: DXP A reads and writes to my liferay-[companyId] index, and :liferay-waffle: DXP B writes to it as well.Hi, my name is :liferay-waffle: DXP B. I can write data to :elasticsearch: ES X, but I have to read it from :elasticsearch: ES Y.Hi, my name is :elasticsearch: ES Y. No client writes to me, I just pull straight just my buddy :elasticsearch: ES X. :liferay-waffle: DXP B can read from me though. I'm the real :star: of this show, because my presence provides the Disaster Recovery and Data Locality LES customers need. (edited) 
DIAGRAM TO CLEARLY ILLUSTRATE

Table from Andre's kickoff to illustrate the differences between multiple data center (unsupported) versus ccr (supported), or to illustrate the differences between a classic single location installation versus a ccr multiple connections situation.

Keep in mind the multiple connections story and how it interacts with ccr

A possible scenario?

Data center 1: Liferay DXP nodes from LiferayCluster, and an Elasticsearch cluster, with a normal remote connection reading and writing from the local Elasticsearch cluster, and accepting write requests from the remote DXP nodes.

Data center 2: Liferay DXP nodes from LiferayCluster, and a separate Elasticsearch cluster, with local-follower indices for reading (from the local DXP servers), and writing to the leader indices in Data Center 1.

To demonstrate, we're going to set up the simplest cluster that gets you the benefits of Cross-cLuster Replication.

DIAGRAM, TWO DXP SERVERS, TWO ES SERVERS, make sure to show that these systems are in two different data centers. The DXP nodes are part of a cluster, while the Elasticsearch servers are separate clusters with a replicated index.

## Install the Cross-Cluster Replication Module

<!-- Does a docker container constitute a single node in a cluster? if so we'll need to instruct to use the mp app in control panel to install the ccr module into each dxp node that will use a read-only connection to the follower cluster. -->

1. Download the Liferay DXP Cross-cluster Replication for Elasticsearch LPKG from the LES downloads page.

1. Install it into the docker image for Liferay DXP instance one.<!-- how do we do this? -->

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

Both clusters we configure will need these indexes.

The most straightforward indexes to deal with are the `liferay-*` indexes.

### Configure Elasticsearch Server 1

The first Elasticsearch cluster is a regular old REMOTE mode cluster: it accepts reads and writes from it's local Liferay DXP node.

Configure its `elasticsearch.yml`:

```json
cluster.name: LiferayElasticsearchCluster_aES
```

Start the server.


If you're just trying this out and don't yet have the proper license, start an Elasticsearch trial:

```bash
curl -XPOST "http://localhost:9200/_license/start_trial?acknowledge=true"
```

You'll see this in your log: 

```bash
[2020-02-26T10:19:36,420][INFO ][o.e.l.LicenseService     ] [user-pc] license [lf263a315-8da3-41f7-8622-lfd7cc14cae29] mode [trial] - valid
```

### Configure Elasticsearch Server 2 

The second Elasticsearch cluster is going to be the follower.

Configure its `elasticsearch.yml`, specifying the `http.port` and `transport.port` to make sure they don't collide with the other Elasticsearch server:

```json
cluster.name: LiferayElasticsearchCluster_bES
http.port: 9202
transport.port: 9500-9600
```

Start the server.

If you're just trying this out and don't yet have the proper license, start an Elasticsearch trial:

```bash
curl -XPOST "http://localhost:9202/_license/start_trial?acknowledge=true"
```

## Configure the Liferay DXP Connectors to Elasticsearch

### Configure DXP 1

Admin prepares Liferay Instance aDXP, or A.
portal-ext.properties
    cluster.link.enabled=true
    cluster.link.channel.name.control=test-control-channel
    cluster.link.channel.name.transport.0=test-transport-channel
    module.framework.properties.osgi.console=11311
Admin prepares Liferay Elasticsearch Configuration on aDXP, or A.
osgi/configs/com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
    logExceptionsOnly="false"
    operationMode="REMOTE"
    clusterName = "LiferayElasticsearchCluster_aES"
(Liferay Elasticsearch Configuration aDXP(A): Cluster is aES(X).)

Where does this read from? Or is this a noraml read/write configuration, and the second connection is a read-only?

Start the DXP server and reindex.

## Set the Leader into the Second Elasticsearch Server

You can use Kibana to run the Elasticsearch Follow API at the Follower Elasticsearch Cluster, to replicate liferay-companyId indexes, the Workflow indexes and any custom indexes you might have. Here we'll show a cURL call that does the same thing: 

First set the cluster running at `ip:port` as the remote leader:

```bash
curl -XPUT "http://localhost:9202/_cluster/settings" -H 'Content-Type: application/json' -d'{  "persistent" : {    "cluster" : {      "remote" : {        "leader" : {          "seeds" : [            "127.0.0.1:9300"           ]        }      }    }  }}'


[2020-02-26T14:15:54,752][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,758][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,759][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,760][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,815][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,819][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,820][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
[2020-02-26T14:15:54,821][INFO ][o.e.c.s.ClusterSettings  ] [russell-pc] updating [cluster.remote.leader.seeds] from [[]] to [["127.0.0.1:9300"]]
```

Now pull, or replicate, the indexes from the leader, using the Follow API:
```bash
curl -XPUT "http://localhost:9202/liferay-20101/_ccr/follow?wait_for_active_shards=1" -H 'Content-Type: application/json' -d'{  "remote_cluster" : "leader",  "leader_index" : "'liferay-20101'"}'

[2020-02-26T14:25:06,806][INFO ][o.e.c.r.a.AllocationService] [russell-pc] Cluster health status changed from [YELLOW] to [GREEN] (reason: [shards started [[liferay-20101][0]]]).
[2020-02-26T14:25:06,867][DEBUG][o.e.a.a.c.s.r.RestoreClusterStateListener] [russell-pc] restore of [_latest_/_latest_] completed
[2020-02-26T14:25:07,019][INFO ][o.e.x.c.a.ShardFollowTasksExecutor] [russell-pc] [liferay-20101][0] Starting to track leader shard [liferay-20101][0]
[2020-02-26T14:25:07,080][INFO ][o.e.x.c.a.ShardFollowNodeTask] [russell-pc] [liferay-20101][0] following leader shard [liferay-20101][0], follower global checkpoint=[177], mapping version=[12], settings version=[0], aliases version=[0]
```

Do the same for all the indexes you see listed at Control Panel &rarr; Configuration &rarr; Search &rarr; Field Mappings. For example, these indexes will be present in most systems:
<!-- Note, we need to talk about why we're not doing this for `liferay-0`, because we don't really ever talk about liferay-0 anywhere else. -->
don't i also need liferay-search-tuning-rankings? my portalB complained when i searched, with a noindexfoundexception on that index

workflow-metrics-instances
workflow-metrics-nodes
workflow-metrics-processes
workflow-metrics-sla-instance-results
workflow-metrics-sla-task-results
workflow-metrics-tokens
liferay-search-tuning-rankings
liferay-0

(Elasticsearch Cluster Y replicates indexes `liferay-companyId` and `workflow-*` from Elasticsearch Cluster X)

### Configure DXP 2

Admin prepares Liferay Instance bDXP, or B.
portal-ext.properties

```properties
cluster.link.enabled=true
cluster.link.channel.name.control=test-control-channel
cluster.link.channel.name.transport.0=test-transport-channel
```

module.framework.properties.osgi.console=11312
Admin prepares Liferay Elasticsearch Configuration on bDXP, or B.
osgi/configs/com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config
    logExceptionsOnly="false"
    operationMode="REMOTE"
    clusterName = "LiferayElasticsearchCluster_aES"
osgi/configs/com.liferay.portal.search.elasticsearch.cross.cluster.replication.internal.configuration.CrossClusterReplicationConfiguration.config

```properties
ccrEnabled="true"
clusterName="LiferayElasticsearchCluster_bES"
transportAddresses="localhost:9500"
```

(Liferay Elasticsearch Configuration bDXP(B): Remote Cluster is aES(X), Local Cluster is bES(Y).)


Admin prepares Liferay Cluster with Instance A and Instance B.
Already prepared the bundles earlier
Once the bundles are preparded, start Liferay Instance A. Reindex into the remote ES server.

(Liferay Instance A creates Liferay index at Cluster X: liferay-20101)
Use Chrome’s Elasticsearch Head plugin to check indexes
Admin uses Kibana to run Elasticsearch Follow API at Elasticsearch Cluster Y to replicate liferay-20101 and the Workflow indexes. 
    curl -XPUT "http://localhost:9202/_cluster/settings" -H 'Content-Type: application/json' -d'{  "persistent" : {    "cluster" : {      "remote" : {        "leader" : {          "seeds" : [            "127.0.0.1:9300"           ]        }      }    }  }}'
//RUN THIS CURL FOR WORKFLOW INDEXES AS WELL
    curl -XPUT "http://localhost:9202/liferay-20101/_ccr/follow?wait_for_active_shards=1" -H 'Content-Type: application/json' -d'{  "remote_cluster" : "leader",  "leader_index" : "'liferay-20101'"}'
(Elasticsearch Cluster Y replicates index liferay-20101 from Elasticsearch Cluster X)
Admin starts Liferay Instance B.
(Liferay Instance B notices already present Liferay index at Cluster Y: liferay-20101)
(Liferay Instance A and Liferay Instance B form a Liferay Cluster.)
User Alice visits Liferay Instance A and creates content K.
(Liferay Instance A indexes content K into Elasticsearch Cluster X.)
User Bob visits Liferay Instance B and creates content L.
(Liferay Instance B indexes content L into Elasticsearch Cluster X, the Leader.)
User Alice searches for contents K and L.
(Liferay Instance A finds K and L at Elasticsearch Cluster X.)
User Bob searches for contents K and L.
(Liferay Instance B finds K and L at Elasticsearch Cluster Y, the Follower.)
So it doesn't matter. What we think of is the geographic separation between ES clusters only. 

aDXP reads AND writes from aES
bDXP writes to aES, reads from bES

The location of your liferay cluster does not matter. assume nothing. only one node matters to consider, because a Liferay cluster has one pre-defined node as its master, and it will coordinate between the search engine and the other nodes. all other dxp nodes route requests through the master, or whatever node is the one that manages the search engine requests.

so, the follower cluster/index (we should assume 1:1 here) is this node's local. the actual location of the remote also does not matter (can be in the same data center as other dxp nodes or not, makes no difference). all write requests from dxp will come to this cluster/node.

what might be tripping me up with ccr is that the example steps show me how to set up 2 clustered dxp nodes, but i can't understand why this matters for ccr. according to the diagram in adnre's initial kickoff presentation (linked below), only one dxp node is communicating with elasticsearch, so it needs to know about 2 connections: the leader that it can write to, and the follower (geographically co-located I guess?) that it can read from. so the simplest demo would be to set up only one dxp node with both connections. do i need to cluster dxp for the docs demo?

going further, would the clustered setup from the example steps even be useful for readers? it appears that one of the node si set up as a normal remote (read/write to one of the es clusters), and the other has the follower as its connection.


LPS-101595 As a Search Admin, I want certain instances in my Liferay cluster to write into a Leader index and read from a Follower index (bundle) 




1) Start up 2 separate elastic servers, each one having a unique cluster name. One cluster is the remote cluster (containing "leader" indexes), the other is the local cluster (containing "follower" indexes).
2) Start a kibana server for each elastic server.

Follow steps 3 through 7 in "Dev Tools" window of Kibana, OR follow steps in these slides

3) Start a trial license in both servers: 
For Elastic 7:

```json
POST /_license/start_trial?acknowledge=true
```

For Elastic 6:

```json
POST _xpack/license/start_trial?acknowledge=true
```

4) In the local cluster, connect to the remote cluster using the alias "leader". Make sure the remote cluster ("seeds") hostname and port is correct.

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

5) In the remote cluster, create an index called "alpha". This is the leader index.
6) In the local cluster, create a follower index called "bravo" by running this command:

```json
PUT /bravo/_ccr/follow?wait_for_active_shards=1
{
  "remote_cluster" : "leader",
  "leader_index" : "alpha"
}
```

7) Index documents into the "alpha" index. Notice that they are automatically replicated in the "bravo" index.

Bonus:

If you connect liferay to the remote cluster, a liferay-12345 index will be created in it.
To create a replica/follower liferay-12345 index in the local cluster, run the Create follower API again:

```json
PUT /liferay-12345/_ccr/follow?wait_for_active_shards=1
{
  "remote_cluster" : "leader",
  "leader_index" : "liferay-12345"
}
```


Document our support for Cross Cluster Replication. See LPS-96663

7.2: The CCR module will provide a Connections tab in Search Admin, won't be present without the CCR module

7.3: The Connections tab will be present regardless of the CCR module's deployment (CE and DXP).


For CCR with ES6: “soft deletes” must be enabled, but for ES6 they are disabled by default
https://www.elastic.co/guide/en/elasticsearch/reference/6.8/ccr-requirements.html

to enable soft deletes on the liferay index, add this line to Additional Index Configurations and then do a full reindex:
index.soft_deletes.enabled: true

https://liferay.slack.com/archives/CNJBT01ME/p1579716939002900

my notes from our demo session w/ bengler
master and 7.2.x difference 
elasticsearch 6 and 7 diff

## 7.2.x
elasticsearchconnection class in same package as the base embedded and remote connection
mirrors the remote connection class
instead of coming from the elasticsearch or xpack security configuration, the security settings come from the ccr config
when activated, the ccr config takes precedence over the es config and the security config
the ccr config has similar properties to the remote config
original es config is the same
if enabled, only the REMOTE operation mode matters from the Elasticsearch 6/7 configuration. everything else must be configured via the ccr config
the same ccr module will work with elasticsearch 6 and 7 depending which connector is deployed
considerations: migrating from es 6 to es 7 with ccr configured
security: the dedicated x-pack security settings apply only to remote mode without ccr
don't use different es 6 and es 7 versions in leader and follower clusters. actually link to es docs on this because some versions can be used this way
all works on the fly? wrapping the configuration in a wrapper-impl class to make sure that when @Modified, the new config is sucked in on the fly.
connections tab only if ccr is installed: still shows the ugly search engine connection info on search admin landing page 
only support one remote and one ccr connection

## 7.3
configure the different configuration in the elasticsearch configuration entry
the elasticsearch configuration is much more barebones now
upgrade: the process will convert your remote configuration and move it to the elasticsearch connections config
upgrade: might be doing the same for 7.2 ccr configurations too, but not yet
when you deactivate a connection, it won't show in the search admin connections section
as many connections as you want are supported
no more ugly search engine string on search admin landing page
the read and write flags in the connections tab tell you which connection is being used for which operation (both read/write for a normal remote, and separate read/write for ccr)

now we have a map of connections, with the connection id being the identifier. call the api, for a specific connection id. no specified id, prefer the local cluster always, search the cross-cluster cluster, the local cluster. if ccr is enabled of course. otherwise return the connection for the remote cluster

portlet with custom cluster, could use low level api and specify the connection id manually.

configuration model listener, cannot create same connection id and updating connections. handles this kind of logic

more docs issues:
now that we have ad-hoc indexes like result rankings etc., there's no re-index with soft deletes option, so you need 5 steps to enable soft deletes on an existing index.
some customer will have the other indexes from rankings and synonyms, do we need it for workflow metrics?
not synonyms actually

if you want the rankings (and metrics?) to work with ccr, you'll need to o

warning: some things will be wiped out with re-index after installing ccr (at least on es 6, not es 7, soft delete is already enabled by default): your synonyms will be wiped out. and, you'll have to do
your data must be in index with soft delete: true to go to ccr. best to use a new index.
recommend moving to es 7 before using ccr in docs
will always have to upgrade ad hoc indexes though

if have synonyms, need to follow manual steps for going to soft delete otherwise can add something to additionalIndexConfigurations and full reindex
as long as there are no read requests on the index, it won't matter.

read from local, write to remote: write to the remote leader (can also read?), read-only from local followers. so es is managing replication from leader to the follower?
