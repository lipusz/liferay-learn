# Configuring CCR: Enabling Soft Deletes on Elasticsearch 6

> Elasticsearch 6 Only

For Elasticsearch 6 versions that support Cross-Cluster Replication (6.7+), you must enable [soft deletes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/ccr-requirements.html) for all existing indexes. This extra hurdle can be avoided if your are on Elasticsearch 7 where soft deletes are enabled by default, so we strongly recommend you to consider [upgrading to Elasticsearch 7](https://help.liferay.com/hc/en-us/articles/360035444872-Upgrading-to-Elasticsearch-7) first before configuring CCR.

## Step 1: Enabling Soft Deletes on the System and Company Indexes

The system (`liferay-0`) and company (`liferay-[companyId]`) indexes can be soft delete-enabled by adding one line to the _Additional Index Configurations_ setting in Control Panel &rarr; Configuration &rarr; System Settings &rarr; Elasticsearch [Version]:
 
```yaml
index.soft_deletes.enabled: true
```

```note::
   Requires to perform a full reindex from the Server Admin in the Control Panel.
```

## Step 2a: Enabling Soft Deletes on App and Custom Indexes in Existing DXP-Elasticsearch Installations

Deployments with existing indexes should follow the steps below.

The app and custom indexes are those not controlled directly by Liferay's Search Framework. They include Liferay DXP app indexes prefixed with `liferay-search-tuning-*` and `workflow-metrics-*`, and your own custom indexes.

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

## Step 2b: Enabling Soft Deletes on App Indexes Using the Overriding LPKG Files Mechanism

You can customize the default index settings of the out-of-the-box Liferay app-driven indexes by leveraging the [overriding LPKG files](https://help.liferay.com/hc/en-us/articles/360028808552-Overriding-lpkg-Files) mechanism. By doing so, you can ensure that when DXP starts up, the leader indexes will be created with the required settings. This can come in handy for new DXP deployments.

```note::
   The steps below assume that you are building a new DXP 7.2 environment and your Leader Elasticsearch cluster node is empty: it should not contain any `liferay-*` or `workflow-*` indexes.
```

We are going to override three JARs from two LPKG files that are bundled with DXP 7.2.

0. Make sure your DXP cluster nodes are not running
1. Go to `Liferay Home/osgi/marketplace`
2. Locate `Liferay Foundation - Liferay Search Tuning - Impl.lpkg`
3. Open with an archive manager and extract the following JAR files into `osgi/marketplace/override` (create a folder called `override` if it does not exist yet):  
3.1 `com.liferay.portal.search.tuning.rankings.web.jar-x.y.z`  
3.2 `com.liferay.portal.search.tuning.synonyms.web.jar-x.y.z`  
4. Locate `Liferay Forms and Workflow - Liferay Portal Workflow - Impl.lpkg`  
5. Open with an archive manager and extract the following JAR file into `osgi/marketplace/override`:  
5.1 `com.liferay.portal.workflow.metrics.service.jar-x.y.z`  
6. Go to `osgi/marketplace/override` and remove the version like `1.0.21` from the name of the files  
7. Open `com.liferay.portal.search.tuning.rankings.web.jar` with an archive manager  
7.1 Navigate to `META-INF/search` and open `liferay-search-tuning-rankings-index.json` with a text editor  
7.2 Edit the `"settings"` snippet and add `"index.soft_deletes.enabled" : true` so it will look like this:
  ```json
	 "settings": {
		    "index.auto_expand_replicas": "0-all",
		    "index.number_of_shards": 1,
		    "index.soft_deletes.enabled" : true
  }
  ```
  7.3 Save the file and let your archive manager re-pack the JAR automatically.  
8. Repeat step 7.) with `com.liferay.portal.search.tuning.synonyms.web.jar`  
9. Repeat step 7.) with `com.liferay.portal.workflow.metrics.service.jar` (the name of the file is `settings.json` and its content and structure may be different from the Search Tuning settings)  
10. Start DXP (Leader node)
 
For consistency, this should be done on all DXP cluster nodes residing in your primary (leader) data center.
 
```note::
    It is recommended to review the default settings files after installing a new patch on your DXP environment and adjust your override files accordingly, if needed.
```
