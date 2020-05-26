# Configuring CCR: Enabling Soft Deletes on Elasticsearch 6

> Elasticsearch 6 Only

For Elasticsearch 6 versions that support Cross-Cluster Replication (6.7+), you must enable [soft deletes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/ccr-requirements.html) for all existing indexes. This extra hurdle can be avoided if your are on Elasticsearch 7 where soft deletes are enabled by default, so you should [upgrade to Elasticsearch 7](https://help.liferay.com/hc/en-us/articles/360035444872-Upgrading-to-Elasticsearch-7) if at all possible before configuring CCR.

## Step 1: Enabling Soft Deletes on the System and Company Indexes

The system (`liferay-0`) and company (`liferay-[companyId]`) indexes can be soft delete-enabled by adding one line to the _Additional Index Configurations_ setting in Control Panel &rarr; Configuration &rarr; System Settings &rarr; Elasticsearch [Version]:
 
```yaml
index.soft_deletes.enabled: true
```

Alternatively, specify the ot delete setting in a configuration file named `com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config` and placed in `[Liferay Home]/osgi/configs`. This file would have the following contents:
```properties
additionalIndexConfigurations = "index.soft_deletes.enabled: true"
```
<!-- To Tibor: You've added this configuration to the "Configuring Cross-Cluster Replication" article. I think it belongs here only. -Russ -->

```note::
   These steps require performing a full reindex from Control Panel &rarr; Configuration &rarr; Search, in the Index Actions tab.
```

<!-- Stopped here: I need to work on the rest of this file still -Russ -->
## Step 2: Enabling Soft Deletes on App Indexes

The app and custom indexes are those not controlled directly by Liferay's Search Framework: they're entirely in control of their own index settings and mappings. They include Liferay DXP app indexes prefixed with `liferay-search-tuning-*` and `workflow-metrics-*`, and your own custom indexes.

There are two approaches for enabling soft deletes on App and Custom Indexes: one that only new deployments can leverage, and one that any deployment, new or existing, can follow.

### New Deployments: Overriding LPKG Files to Enable Soft Deletes

<!-- As written I think this introduces confusion. If this approach is preferable for new deployments, we should just state up front, do this if you have a new deployment, otherwise see -->
You can customize the default index settings of the out-of-the-box Liferay app-driven indexes by leveraging the [overriding LPKG files](https://help.liferay.com/hc/en-us/articles/360028808552-Overriding-lpkg-Files) mechanism. By doing so, you can ensure that when DXP starts up, the leader indexes will be created with the required settings. This can come in handy for new DXP deployments.

```note::
   The steps below assume that you are building a new DXP 7.2 environment and your Leader Elasticsearch cluster node is empty: it should not contain any `liferay-*` or `workflow-*` indexes.
```

We are going to override three JARs from two LPKG files that are bundled with DXP 7.2.

1. Stop your Liferay DXP cluster nodes if they're running.

1. Go to `[Liferay Home]/osgi/marketplace` and find `Liferay Foundation - Liferay Search Tuning - Impl.lpkg`.

1. Open the LPKG with an archive manager and extract the following JAR files into `Liferay Home/osgi/marketplace/override` (create the `override` folder if it does not exist):  

   - `com.liferay.portal.search.tuning.rankings.web.jar-x.y.z`  
   - `com.liferay.portal.search.tuning.synonyms.web.jar-x.y.z`  

1. Next locate the `Liferay Forms and Workflow - Liferay Portal Workflow - Impl.lpkg`  

1. Open the LPKG with an archive manager and extract the following JAR file into `Liferay Home/osgi/marketplace/override`:  

   - `com.liferay.portal.workflow.metrics.service.jar-x.y.z`  

1. Go to `Liferay Home/osgi/marketplace/override` and remove the version (e.g., `1.0.21`) from the name of all the JAR files  

1. Open `com.liferay.portal.search.tuning.rankings.web.jar` with an archive manager 

1. Navigate to `META-INF/search` and open `liferay-search-tuning-rankings-index.json` with a text editor. 

1. Edit the `"settings"` snippet and add `"index.soft_deletes.enabled" : true`:

  ```json
	 "settings": {
		    "index.auto_expand_replicas": "0-all",
		    "index.number_of_shards": 1,
		    "index.soft_deletes.enabled" : true
  }
  ```
1. Save the file and let your archive manager re-package the JAR automatically.  

1. Do the same for `com.liferay.portal.search.tuning.synonyms.web.jar`  

1. Do the same for `com.liferay.portal.workflow.metrics.service.jar` (the name of the file is `settings.json` and its content and structure may be different from the Search Tuning settings).  

1. Start Liferay DXP (the Leader node).
 
For consistency, this should be done on all DXP cluster nodes residing in your primary (leader) data center.
 
```note::
    You should review the default settings files each time you install a new patch in your DXP environment, and adjust your override files accordingly.
```

### Existing Deployments: Manually Enabling Soft Deletes

Deployments with existing App Indexes must follow these steps to enable soft deletes.

<!-- I made it here, will continue working on this file later - Russ -->
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
