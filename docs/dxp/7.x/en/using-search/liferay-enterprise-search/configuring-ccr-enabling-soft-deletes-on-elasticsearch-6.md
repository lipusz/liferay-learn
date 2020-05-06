# Configuring CCR: Enabling Soft Deletes on Elasticsearch 6

> Elasticsearch 6 Only

For Elasticsearch 6 versions that support Cross-Cluster Replication (6.7+), you must enable [soft deletes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/ccr-requirements.html) for all existing indexes. Elasticsearch 7 enables soft deletes by default.

## Enabling Soft Deletes on the System and Company Indexes

The system (`liferay-0`) and company (`liferay-[companyId]`) indexes can be soft delete-enabled by adding one line to the _Additional Index Configurations_ setting in Control Panel &rarr; Configuration &rarr; System Settings &rarr; Elasticsearch [Version]:
 
```yaml
index.soft_deletes.enabled: true
```

## Enabling Soft Deletes on App and Dev Indexes

The app and dev indexes are those not controlled directly by Liferay's Search Framework. They include Liferay DXP app indexes prefixed with `liferay-search-tuning-*` and `workflow-metrics-*`, and your own custom dev indexes.

To enable soft delete manually,

1. Create a temporary (empty, at first) index containing the current mapping.

   You can get the mappings with the [`mappings` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-get-mapping.html) 

   <!-- https://github.com/liferay/liferay-portal/blob/master/modules/dxp/apps/portal-search-tuning/portal-search-tuning-rankings-web/src/main/resources/META-INF/search/liferay-search-tuning-rankings-index.json -->

1. Use the [`reindex` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/docs-reindex.html) to copy the existing data into the temporary index.

1. Delete the original index.

1. Recreate the index, using the above mappings and the soft delete setting.

1. Use the `reindex` API to copy the existing data into the new index.
 
```json
   PUT index
   {
     "settings" : {
       "index.soft_deletes.enabled" : true
     },
     "mappings": {
   		"_doc": {
   			"properties": {
   				"aliases": {
   					"store": true,
   					"type": "text"
   				},
   				"blocks": {
   					"store": true,
   					"type": "keyword"
   				},
   				"inactive": {
   					"store": true,
   					"type": "boolean"
   				},
   				"index": {
   					"store": true,
   					"type": "keyword"
   				},
   				"name": {
   					"fields" : {
   						"keyword" : {
   							"type" : "keyword"
   						}
   					},
   					"store": true,
   					"type": "text"
   				},
   				"pins" : {
   					"properties" : {
   						"position" : {
   							"store": true,
   							"type" : "integer"
   						},
   						"uid" : {
   							"store": true,
   							"type" : "keyword"
   						}
   					},
   					"type" : "nested"
   				},
   				"queryString": {
   					"fields" : {
   						"keyword" : {
   							"type" : "keyword"
   						}
   					},
   					"store": true,
   					"type": "text"
   				},
   				"queryStrings": {
   					"fields" : {
   						"keyword" : {
   							"type" : "keyword"
   						}
   					},
   					"store": true,
   					"type": "text"
   				},
   				"uid": {
   					"store": true,
   					"type": "keyword"
   				}
   			}
   		}
   	}
   }
```

Once your index is in good shape, you're ready to [configure Cross-Cluster Replication](./configuring-cross-cluster-replication.md) for Liferay DXP.

