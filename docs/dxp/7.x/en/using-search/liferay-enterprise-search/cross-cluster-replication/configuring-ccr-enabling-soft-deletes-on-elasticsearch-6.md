# Configuring CCR: Enabling Soft Deletes on Elasticsearch 6

> Elasticsearch 6 Only

For Elasticsearch 6 versions that support Cross-Cluster Replication (6.7+), you must enable [soft deletes](https://www.elastic.co/guide/en/elasticsearch/reference/6.7/ccr-requirements.html) for all existing indexes. This extra hurdle can be avoided if your are on Elasticsearch 7 where soft deletes are enabled by default, so you should [upgrade to Elasticsearch 7](https://help.liferay.com/hc/en-us/articles/360035444872-Upgrading-to-Elasticsearch-7) if at all possible before configuring CCR.

## Enabling Soft Deletes on the System and Company Indexes

Enable soft deletes on the system (`liferay-0`) and company (`liferay-[companyId]`) indexes by adding one line to the _Additional Index Configurations_ setting in Control Panel &rarr; Configuration &rarr; System Settings &rarr; Elasticsearch [Version]:
 
```yaml
index.soft_deletes.enabled: true
```

Alternatively, enable soft deletes in a configuration file named `com.liferay.portal.search.elasticsearch6.configuration.ElasticsearchConfiguration.config` and placed in `[Liferay Home]/osgi/configs`. Give the file these contents:

```properties
additionalIndexConfigurations="index.soft_deletes.enabled: true"
```

```note::
   These steps require performing a full reindex from Control Panel > Configuration > Search, in the Index Actions tab.
```

## Enabling Soft Deletes on App Indexes

The app and custom indexes are those not controlled directly by Liferay's Search Framework: they're entirely in control of their own index settings and mappings. They include Liferay DXP app indexes prefixed with `liferay-search-tuning-*` and `workflow-metrics-*`, and your own custom indexes.

There are two approaches for enabling soft deletes on App and Custom Indexes: one that only new deployments can leverage, and one that any deployment, new or existing, can follow.

## New Deployments: Overriding LPKG Files to Enable Soft Deletes

<!-- As written I think this introduces confusion. If this approach is preferable for new deployments, we should just state up front, do this if you have a new deployment, otherwise see -->
You can customize the default index settings of the out-of-the-box Liferay app-driven indexes by leveraging the [overriding LPKG files](https://help.liferay.com/hc/en-us/articles/360028808552-Overriding-lpkg-Files) mechanism. By doing so, you can ensure that when DXP starts up, the leader indexes will be created with the required settings. This can come in handy for new DXP deployments.

```note::
   The steps below assume that you are building a new DXP 7.2 environment and your Leader Elasticsearch cluster node is empty: it should not contain any ``liferay-*`` or ``workflow-*`` indexes.
```

Override three JARs from two LPKG files that are bundled with Liferay DXP 7.2:

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
    You should review the default settings files each time you `install a new patch <./../../installation-and-upgrades/maintaining-a-liferay-dxp-installation/installing-patches/introduction-to-installing-patches.md>`__ in your DXP environment, and adjust your override files accordingly.
```

## Existing Deployments: Manually Enabling Soft Deletes

Deployments with existing App Indexes must follow these steps to enable soft deletes.

### Enabling Soft Deletes on Search Tuning Result Rankings Index

1. Using the steps above, extract the `liferay-search-tuning-rankings-index.json` file from the Search Tuning LPKG archive's `com.liferay.portal.search.tuning.rankings.web.jar-x.y.z`. This file defines the mappings and settings for the index `liferay-search-tuning-rankings`.

1. The extracted file's JSON provides the `"mappings"` and the `"settings"` you'll use to [create a temporary](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-create-index.html) (empty, at first) index, for example called `liferay-search-tuning-rankings_backup`:

    ```json
    PUT liferay-search-tuning-rankings_backup
    // Copy the exact contents of liferay-search-tuning-rankings-index.json here.
    ```

1. Use the [`_reindex` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/docs-reindex.html) to copy the existing data into the temporary index:

   ```json
   POST _reindex
   {
     "source": {
       "index": "liferay-search-tuning-rankings"
     },
     "dest": {
       "index": "liferay-search-tuning-rankings_backup"
     }
   }
   ```
1. [Delete](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-delete-index.html) the original index:

   ```json
   DELETE /liferay-search-tuning-rankings
   ```

1. [Recreate the index](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-create-index.html), using the default mappings and settings you obtained in the first step plus the soft deletes setting. Use the original index name (`liferay-search-tuning-rankings`):

    ```json
    PUT liferay-search-tuning-rankings_backup
    {
        // Copy the exact contents of liferay-search-tuning-rankings-index.json here.
        // Add the soft deletes configuration to the end of the settings definition.
        "index.soft_deletes.enabled": true
    }
    ```

1. Use the `_reindex` API to copy the existing data into the new index:

    ```json
    POST _reindex
    {
     "source": {
       "index": "liferay-search-tuning-rankings_backup"
     },
     "dest": {
       "index": "liferay-search-tuning-rankings"
     }
    }
    ```
 
7. Delete the temporary index:

    ```json
    DELETE /liferay-search-tuning-rankings_backup
    ```

### Enabling Soft Deletes on the Search Tuning: Synonyms Index

Repeat the steps for Result Rankings, but use the mappings and settings from `com.liferay.portal.search.tuning.synonyms.web.jar` (in the `liferay-search-tuning-synonyms-index.json` file).

### Enabling Soft Deletes on Workflow Metrics Indexes

The Workflow Metrics indexes work differently from the Search Tuning indexes. Six indexes must be soft-delete enabled.

1. Extract the two JSON files from `com.liferay.portal.workflow.metrics.service.jar` (embedded in the Liferay Portal Workflow LPKG):

    - `META-INF/search/mappings.json` 
    - `META-INF/search/settings.json`

    The `mappings` file contains separate mappings sections for each index. The `settings` file is shared; the Workflow Metrics indexes use identical settings.
       
1. Locate the relevant block from the `mappings.json` for each Workflow Metrics index:

    - workflow-metrics-instances: `"WorkflowMetricsInstanceType": { ...}`
    - workflow-metrics-nodes: `"WorkflowMetricsNodeType": {...}`
    - workflow-metrics-processes: `"WorkflowMetricsProcessType": {...}`
    - workflow-metrics-sla-instance-results: `"WorkflowMetricsSLAProcessResultType": {...}`
    - workflow-metrics-sla-task-results: `"WorkflowMetricsSLATaskResultType": {...}`
    - workflow-metrics-tokens: `"WorkflowMetricsTokenType": {...}`

1. [Create a temporary](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-create-index.html) (empty, at first) index, for example called `workflow-metrics-instances_backup`. To specify the `"mappings"` use the relevant mappings block identified above; to specify the `"settings"` use the contents of `settings.json`, like this:

    ```json
    PUT workflow-metrics-instances_backup
    {
      "mappings": {
        "WorkflowMetricsInstanceType": {
        // ...
        },
      },
      "settings": 
      // Contents of settings.json goes here
    }  
    ```

1. Use the [`_reindex` API](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/docs-reindex.html) to copy the existing data into the temporary index.

    ```json
    POST _reindex
    {
     "source": {
       "index": "workflow-metrics-instances"
     },
     "dest": {
       "index": "workflow-metrics-instances_backup"
     }
    }
    ```

1. [Delete](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-delete-index.html) the original index.

1. [Recreate the index](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-create-index.html), using the original index name (`workflow-metrics-instances`):

   Repeat step 3 (of course, change the index name) and also add `"index.soft_deletes.enabled" : true` to the end of the `"settings"` definition to enable soft deletes. 

1. [Recreate the index](https://www.elastic.co/guide/en/elasticsearch/reference/6.x/indices-create-index.html), using the mappings and settings as the backup index, plus the soft deletes setting. Use the original index name (e.g., `workflow-metrics-instances`):

    ```json
    PUT workflow-metrics-instances
    {
        // Copy in the same mappings and settings used to create the backup. 
        // Add the soft deletes configuration to the end of the settings definition.
        "index.soft_deletes.enabled": true
    }
    ```

1. Use the `_reindex` API to copy the existing data into the new index. 

    ```json
    POST _reindex
    {
     "source": {
       "index": "workflow-metrics-instances_backup"
     },
     "dest": {
       "index": "workflow-metrics-instances"
     }
    }
    ```

1. Delete the temporary index (e.g., `workflow-metrics-instances_backup`).

1. Follow these steps for each additional Workflow Metrics index.

Once your Elasticsearch 6 indexes are soft-delete enabled, you're ready to [configure Cross-Cluster Replication](./configuring-cross-cluster-replication.md) for Liferay DXP.
