# Troubleshooting Cross-Cluster Replication

Known common pitfalls encountered during the CCR setup are covered here, as well as general troubleshooting techniques. For further troubleshooting, look at [Elastic's CCR documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/ccr-overview.html) or visit [Elastic's forum](https://discuss.elastic.co/tag/cross-cluster-replication).

## Investigating Index Replication Issues

Successful read operations to the follower Elasticsearch cluster depend on replication of the leader's indexes.

To aid diagnosing replication problems, add an INFO log level for `com.liferay.portal.search.elasticsearch7.internal.ccr.CrossClusterReplicationHelperImpl`. Log Levels are added in Control Panel &rarr; Server Administration &rarr; Log Levels.

## Inspecting Connection Request/Response

Enabling Cross-Cluster Replication requires setting up multiple connections to Elasticsearch clusters.

To aid diagnosing connection issues, add an INFO log level for `com.liferay.portal.search.elasticsearch7.internal.connection.ElasticsearchConnectionManager`. Log Levels are added in Control Panel &rarr; Server Administration &rarr; Log Levels.

## Exceptions During Reindex: `RetentionLeaseNotFoundException` and `IndexNotFoundException`

When a reindex is triggered on the Leader DXP node, the Follower Elasticsearch node may throw errors like this:

```bash
[2020-05-25T14:49:21,478][WARN ][o.e.x.c.a.ShardFollowNodeTask] [es-follower-node-1] shard follow task encounter non-retryable error
org.elasticsearch.transport.RemoteTransportException: [es-leader-node-1][127.0.0.1:9300][indices:data/read/xpack/ccr/shard_changes]
Caused by: org.elasticsearch.index.IndexNotFoundException: no such index
  at org.elasticsearch.cluster.routing.RoutingTable.shardRoutingTable(RoutingTable.java:119) ~[elasticsearch-6.8.6.jar:6.8.6]
```

and this:
```bash
[2020-05-25T14:49:50,750][WARN ][o.e.x.c.a.ShardFollowTasksExecutor] [es-follower-node-1] [liferay-20101][0] background management of retention lease [LiferayElasticsearchCluster_FOLLOWER/liferay-20101/3a22HGCGS9iDl5rCbutNHg-following-leader/liferay-20101/lZThZJuhTLSaNYTSxmeX8A] failed while following
org.elasticsearch.index.seqno.RetentionLeaseNotFoundException: retention lease with ID [LiferayElasticsearchCluster_FOLLOWER/liferay-20101/3a22HGCGS9iDl5rCbutNHg-following-leader/liferay-20101/lZThZJuhTLSaNYTSxmeX8A] not found
  at org.elasticsearch.index.seqno.ReplicationTracker.renewRetentionLease(ReplicationTracker.java:282) ~[elasticsearch-6.8.6.jar:6.8.6]
```

From <https://www.elastic.co/blog/follow-the-leader-an-introduction-to-cross-cluster-replication-in-elasticsearch>:

> With a shard history retention lease, a follower can mark in the history of operations on the leader where in history that follower currently is. The leader shards know that operations below that marker are safe to be merged away, but any operations above that marker must be retained for until the follower has had the opportunity to replicate them. These markers ensure that if a follower goes offline temporarily, the leader will retain operations that have not yet been replicated. Since retaining this history requires additional storage on the leader, these markers are only valid for a limited period after which the marker will expire and the leader shards will be free to merge away history. You can tune the length of this period based on how much additional storage you are willing to retain in case a follower goes offline, and how long you’re willing to accept a follower being offline before it would otherwise have to be re-bootstrapped from the leader.

## `ElasticsearchSecurityException` When Setting Up CCR

You may run into the following error when configuring CCR:

```bash
ElasticsearchSecurityException security_exception current license is non-compliant for [ccr]
```

[CCR requires](https://www.elastic.co/subscriptions#scalability-&-resiliency) a Platinum Elasticsearch license. As a LES subscriber you have access to CCR with the license provided to you by Liferay.

## Clustered DXP Nodes Won't Read from Multiple Local/Follower Elasticsearch Clusters

In a DXP cluster using Cross-Cluster Replication, each node can be mapped to read from a dedicated local Elasticsearch cluster. To match each DXP node with an Elasticsearch cluster, the CCR Local Cluster Connection Configurations property is configured with values like this:

```properties
localhost:9080,follower1
localhost:9180,follower2
```

Even if you're not binding the DXP nodes to `localhost`, the internal clustering code continues to identify each node using it; so `localhost` should be the hostname in this property. If you want to use a hostname other than `localhost` to identify DXP nodes internally (including in the CCR configuration) you must set the following [portal properties](./../../../installation-and-upgrades/reference/portal-properties.md) on each DXP node:

```properties
portal.instance.protocol=http
portal.instance.inet.socket.address=myhostname:9080
```

## Follower Cluster with Status Red Indexes

After you successfully set up a CCR connection and your leader indexes are replicated successfully into the follower cluster, you can see some indexes go to the [red status](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/cluster-health.html). This may happen if you've been configuring, restarting, and re-indexing repeatedly throughout the setting up procedure. If you see this happen and you are confident your connections are configured properly, delete the follower indexes, then re-enable the LES Cross-Cluster-Replication configuration to trigger a fresh replication and following of the leader indexes. 

1. Delete all the follower indexes. This is most conveniently carried out in Kibana's Index Management UI.

2. To re-enable the CCR configuration, go to System Settings &rarr; Search &rarr; Cross-Cluster Replication. De-select _Enabled_ and click _Update_ to disable the module, then select _Enabled_ and click _Update_ again to re-enable it. 
