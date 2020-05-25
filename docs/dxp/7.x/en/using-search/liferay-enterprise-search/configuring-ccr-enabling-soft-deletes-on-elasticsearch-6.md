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

You can also specify the additional setting by creating a file named `com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config` in `[Liferay Home]/osgi/configs` with the following content:
```properties
additionalIndexConfigurations = "index.soft_deletes.enabled: true"
```

## Step 2: Enabling Soft Deletes on App Indexes

The app and custom indexes are those not controlled directly by Liferay's Search Framework in terms of index settings and mappings. They include Liferay DXP app indexes prefixed with `liferay-search-tuning-*` and `workflow-metrics-*`, and your own custom indexes.

### Step 2a: Enabling Soft Deletes on App Indexes Using the Overriding LPKG Files Mechanism

You can customize the default index settings of the out-of-the-box Liferay app-driven indexes by leveraging the [overriding LPKG files](https://help.liferay.com/hc/en-us/articles/360028808552-Overriding-lpkg-Files) mechanism. By doing so, you can ensure that when DXP starts up, the leader indexes will be created with the required settings. This can come in handy for new DXP deployments.

```note::
   The steps below assume that you are building a new DXP 7.2 environment and your Leader Elasticsearch cluster node is empty: it should not contain any `liferay-*` or `workflow-*` indexes.
```

We are going to override three JARs from two LPKG files that are bundled with DXP 7.2.

0. Make sure your DXP cluster nodes are not running
1. Go to `[Liferay Home]/osgi/marketplace`
2. Locate `Liferay Foundation - Liferay Search Tuning - Impl.lpkg`
3. Open with an archive manager and extract the following JAR files into `osgi/marketplace/override` (create a folder called `override` if it does not exist yet):  
3.1 `com.liferay.portal.search.tuning.rankings.web.jar-x.y.z`  
3.2 `com.liferay.portal.search.tuning.synonyms.web.jar-x.y.z`  
4. Locate `Liferay Forms and Workflow - Liferay Portal Workflow - Impl.lpkg`  
5. Open with an archive manager and extract the following JAR file into `osgi/marketplace/override`:  
5.1 `com.liferay.portal.workflow.metrics.service.jar-x.y.z`  
6. Go to `osgi/marketplace/override` and remove the version like `1.0.21` from the name of the files  
7. Open `com.liferay.portal.search.tuning.rankings.web.jar` with an archive manager  
7.1. Navigate to `META-INF/search` and open `liferay-search-tuning-rankings-index.json` with a text editor  
7.2. Edit the `"settings"` snippet and add `"index.soft_deletes.enabled" : true` so it will look like this:
  ```json
	 "settings": {
		    "index.auto_expand_replicas": "0-all",
		    "index.number_of_shards": 1,
		    "index.soft_deletes.enabled" : true
  }
  ```
   7.3. Save the file and let your archive manager re-pack the JAR automatically.  
8. Repeat step 7.) with `com.liferay.portal.search.tuning.synonyms.web.jar`  
9. Repeat step 7.) with `com.liferay.portal.workflow.metrics.service.jar` (the name of the file is `settings.json` and its content and structure may be different from the Search Tuning settings)  
10. Start DXP (Leader node)
 
For consistency, this should be done on all DXP cluster nodes residing in your primary (leader) data center.
 
```note::
    It is recommended to review the default settings files after installing a new patch on your DXP environment and adjust your override files accordingly, if needed.
```

### Step 2b: Enabling Soft Deletes on Existing App Indexes

Deployments with existing indexes should follow the steps below.

To enable soft delete manually,

#### Enabling Soft Deletes on Search Tuning Result Rankings Index

1. Follow steps 1-7.) in "Step 2a" above to obtain `liferay-search-tuning-rankings-index.json` for index `liferay-search-tuning-rankings`
2. Use the content of the JSON to specify the `"mappings"` and the `"settings"` and [create a temporary](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-create-index.html) (empty, at first) index, for example called `liferay-search-tuning-rankings_backup`, like this:
    ```json
    PUT liferay-search-tuning-rankings_backup
    // Content of liferay-search-tuning-rankings-index.json goes here as-is.
    ```
3. Use the [`_reindex` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/docs-reindex.html) to copy the existing data into the temporary index.
    `liferay-search-tuning-rankings` -> `liferay-search-tuning-rankings_backup`
4. [Delete](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-delete-index.html) the original index.
5. [Recreate the index](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-create-index.html), using the default mappings and settings you obtained in the first step, using the original index name (`liferay-search-tuning-rankings`):

   Add `"index.soft_deletes.enabled" : true` to the end of the `"settings"` definition to enable soft deletes.

6. Use the `_reindex` API to copy the existing data into the new index. 

      `liferay-search-tuning-rankings_backup` -> `liferay-search-tuning-rankings`
 
7. Delete the temporary index (`liferay-search-tuning-rankings_backup`).

#### Enabling Soft Deletes on Search Tuning Synonyms Index

Same as Result Rankings, but use `com.liferay.portal.search.tuning.synonyms.web.jar` to obtain the default settings JSON (`liferay-search-tuning-synonyms-index.json`).

#### Enabling Soft Deletes on Workflow Metrics Indexes

It is slightly different from the Search Tuning index steps, because the Workflow Metrics consists of six indexes.

1. When inspecting `com.liferay.portal.workflow.metrics.service.jar` you will find 2 JSON files:

      `META-INF/search/mappings.json` and `META-INF/search/settings.json`. Obtain both.
       
2. Locate the relevant block from the `mappings.json` for each Workflow Metrics index:
   - workflow-metrics-instances: `"WorkflowMetricsInstanceType": { ...}`
   - workflow-metrics-nodes: `"WorkflowMetricsNodeType": {...}`
   - workflow-metrics-processes: `"WorkflowMetricsProcessType": {...}`
    - workflow-metrics-sla-instance-results: `"WorkflowMetricsSLAProcessResultType": {...}`
    - workflow-metrics-sla-task-results: `"WorkflowMetricsSLATaskResultType": {...}`
    - workflow-metrics-tokens: `"WorkflowMetricsTokenType": {...}`
3. [Create a temporary](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-create-index.html) (empty, at first) index, for example called `workflow-metrics-instances_backup`. To specify the `"mappings"` use the block you identified in step 2; to specify the `"settings"` use the content of `settings.json`, like this:
```json
	PUT workflow-metrics-instances_backup
	{
	  "mappings": {
	    "WorkflowMetricsInstanceType": {
	    // ...
	    },
	  },
	  "settings": 
	  // Content of settings.json goes here as-is.
	}  
```
4. Use the [`_reindex` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/docs-reindex.html) to copy the existing data into the temporary index.
    `workflow-metrics-instances` -> `workflow-metrics-instances_backup`
5. [Delete](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-delete-index.html) the original index.
6. [Recreate the index](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-create-index.html), using the original index name (`workflow-metrics-instances`):

   Repeat step 3 (of course, change the index name) and also add `"index.soft_deletes.enabled" : true` to the end of the `"settings"` definition to enable soft deletes. 

7. Use the `_reindex` API to copy the existing data into the new index. 

    `workflow-metrics-instances_backup` -> `workflow-metrics-instances`

8. Delete the temporary index (`workflow-metrics-instances_backup`).

9. Repeat steps 3-8 with the other Workflow Metrics indexes.

Once your indexes are in good shape, you're ready to [configure Cross-Cluster Replication](./configuring-cross-cluster-replication.md) for Liferay DXP.
