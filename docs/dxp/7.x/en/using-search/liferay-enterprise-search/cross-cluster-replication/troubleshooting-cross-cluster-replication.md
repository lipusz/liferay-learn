# Troubleshooting Cross-Cluster Replication

<!--add https://issues.liferay.com/browse/LPS-113128 (company and system indexes occasionally not replicated after reindex, the workaround is to manually replicate again -->

Known common pitfalls encountered during the CCR setup are covered here. For further troubleshooting, look at [Elastic's CCR documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/ccr-overview.html) or visit [Elastic's forum](https://discuss.elastic.co/tag/cross-cluster-replication).

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

## ElasticsearchSecurityException When Setting Up CCR

You may run into the following error when configuring CCR:

```bash
ElasticsearchSecurityException security_exception current license is non-compliant for [ccr]
```

[CCR requires](https://www.elastic.co/subscriptions#scalability-&-resiliency) a Platinum Elasticsearch license. As a LES subscriber you are allowed to use CCR.

## SnapshotRestoreException on the Follower Elasticsearch Node During Reindex

If you set up an [auto-follow pattern](./configuring-ccr-in-a-local-follower-data-center.md#replicate-the-leader-indexes) to follow the leader indexes when [configuring the local follower Elasticsearch cluster](./configuring-ccr-in-a-local-follower-data-center.md#configuring-auto-follow) and it is still in use when you reindex, you can get a similar error in the Follower Elasticsearch node's console:

```bash
[2020-05-28T14:25:30,973][WARN ][o.e.s.RestoreService] [es-follower-node-1] [_latest_/_latest_] failed to restore snapshot
org.elasticsearch.snapshots.SnapshotRestoreException: [_ccr_leader:_latest_/_latest_] cannot restore index [liferay-20101] because an open index with same name already exists in the cluster. Either close or delete the existing index or restore the index under a different name by providing a rename pattern and replacement name
```

This is happening because system and company indexes (`liferay-0` and `liferay-<companyId>`) are re-followed automatically when a reindex is performed in Liferay DXP. Since the app-driven indexes like the Search Tuning and Workflow Metrics indexes will not be dropped and re-created by a full reindex, it's safe to delete the auto-follow patterns after the initial installation is done.
