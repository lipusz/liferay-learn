# Upgrading to Elasticsearch 7.9

Elasticsearch 7.9+ is supported for Liferay 7.3 out of the box. For Liferay 7.2 it's supported via the [Liferay Connector to Elasticsearch 7](https://web.liferay.com/marketplace/-/mp/application/170390307) (version 3.1.0+). If you're upgrading to Liferay 7.3 or wanting to bring your existing 7.2 search engine to Elasticsearch 7.9+, you must upgrade your Elasticsearch servers. If you're setting up a new Liferay 7.2 system, install Elasticsearch 7.9 and follow the [installation guide](../getting-started-with-elasticsearch.md).

```important::
   Before upgrading Elasticsearch, back up your existing data. If something goes wrong during or after the upgrade, roll back to the previous version using the uncorrupted index snapshots. Follow the steps in `Backing up Elasticsearch <./backing-up-elasticsearch.md>`__.
```

To upgrade an existing Elasticsearch server (or cluster) to Elasticsearch 7.9+,

1. [Back up the Liferay Company and System Indexes](./backing-up-elasticsearch.md).

1. [Back up the application specific indexes for Search Tuning](#backing-up-and-restoring-search-tuning-indexes) (Synonym Sets and Result Rankings).

1. [Install and configure Elasticsearch 7.9](../installing-elasticsearch.md).

1. [Upgrade Elasticsearch](#upgrading-elasticsearch)

1. If you're using X-Pack security, make sure it's enabled:

   ```yaml
   xpack.security.enabled: true
   ```

   See [Securing Elasticsearch](../securing-elasticsearch.md) for detailed coverage of the security configuration.

1. \[7.2 only\] [Blacklist the bundled Liferay Connector to Elasticsearch 6](#blacklisting-elasticsearch-6).

1. [Install](../connecting-to-elasticsearch.md#install-the-elasticsearch-7-connector) and configure the Liferay Connector to Elasticsearch 7.

1. Re-index all search and spell check indexes.

1. Verify that Search Tuning entries have been carried over

**Known Issue:** See [LPS-103938](https://issues.liferay.com/browse/LPS-103938). The Liferay Connector to Elasticsearch 7 throws an exception in the log when the LPKG file is deployed. There are no known functional impacts. If unexpected errors occur, re-start the @product@ server.

## Upgrading Elasticsearch

<!-- does this make sense here? Does it apply when going from 6.x to 7.9+ -->
If you are using a rolling-restart eligible version (`6.8.x`), doing a [rolling upgrade](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/rolling-upgrades.html) is the recommended way to ugprade your Elasticsearch cluster. Otherwise, follow the [full cluster restart upgrade
](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/restart-upgrade.html) guide.

## Blacklisting Elasticsearch 6

To blacklist Elasticsearch 6,

1.  Create a configuration file named

    ```bash
    com.liferay.portal.bundle.blacklist.internal.BundleBlacklistConfiguration.config
    ```

1.  Give it these contents:

    ```properties
    blacklistBundleSymbolicNames=[ \
        "com.liferay.portal.search.elasticsearch6.api", \
        "com.liferay.portal.search.elasticsearch6.impl", \
        "com.liferay.portal.search.elasticsearch6.spi", \
        "com.liferay.portal.search.elasticsearch6.xpack.security.impl", \
        "Liferay Connector to X-Pack Security [Elastic Stack 6.x] - Impl", \ 
        "Liferay Enterprise Search Security  - Impl" \
    ]
    ```
<!--Does it make sense to blacklist the LES Security app?-->

## Re-index

Once Liferay is connected with the Elasticsearch cluster, re-index the applicable indexes into the new Elasticsearch installation:

1. Re-index the company, system, and spell check indexes: from the Global Menu (![Global Menu](../../../../images/icon-applications-menu.png)),navigate to *Control Panel* &rarr; *Configuration* &rarr; *Search*. Click *Execute* for the *Reindex all search indexes* entry.

1. Re-index the [Workflow Metrics indexes](../../../../process-automation/workflow/user-guide/workflow-metrics-reports.html#re-indexing-workflow-metrics): from the Global Menu (![Global Menu](../../../../images/icon-applications-menu.png)), navigate to *Applications* &rarr; *Workflow---Metrics*. Open the Settings menu (![Options](../../../../images/icon-options.png) and Click *Reindex All*.

## Reverting to Elasticsearch 6

Stuff happens. If that stuff involves an unrecoverable failure during the upgrade to Elasticsearch 7, roll back to Elasticsearch 6 and regroup.

Since your Elasticsearch 6 and 7 are currently two separate installations, this procedure takes only a few steps:

1.  Stop the Liferay Connector to Elasticsearch 7.

1.  Stop Elasticsearch 7 and make sure that the Elasticsearch 6 `elasticsearch.yml` and the connector app are configured to use the same port (9200 by default).

1.  Start the Elasticsearch server, and then restart the Liferay Connector to Elasticsearch 7.

## Backing up and Restoring Search Tuning Indexes

Creating a snapshot of your Elasticsearch indexes is highly recommended, especially for indexes that act as the primary storage format: for example, the Search Tuning features ([Synonym Sets](../../../search_administration_and_tuning.rst) and [Result Rankings](../../../search_administration_and_tuning.rst)). There are no records for these applications in the database.

You can use Elasticsearch's [snapshot and restore](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/snapshot-restore.html) feature to back up and restore the Search Tuning indexes.

1. Create a folder called `elasticsearch_local_backup` somewhere in the system. Make sure Elasticsearch has read and write access tothe folder (e.g., `/path/to/elasticsearch_local_backup`).

1. Add 

    ```yaml
    path.repo: [ "/path/to/elasticsearch_local_backup" ]
    ```

   to the `elasticsearch.yml` for [all master and data nodes](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/snapshots-register-repository.html#snapshots-filesystem-repository) in the Elasticsearch cluster.

1. Restart all Elasticsearch nodes.

1. [Register the snapshot repository](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/snapshots-register-repository.html). You can run the following `snapshot` API request (for example through the Dev Tools console in Kibana):

    ```json
    PUT /_snapshot/elasticsearch_local_backup
    {
      "type": "fs",
      "settings": {
        "location": "/path/to/elasticsearch_local_backup"
      }
    }
    ```

1. [Create a snapshot](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/snapshots-take-snapshot.html):

    ```json
    PUT /_snapshot/elasticsearch_local_backup/snapshot1?wait_for_completion=true
    {
      "indices": "liferay-search-tuning*",
      "ignore_unavailable": true,
      "include_global_state": false
    }
    ```

    If you want to create a snapshot for all Liferay indexes, you can use `"indices": "liferay*,workflow-metrics*"` instead.

1. To [restore](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/snapshots-restore-snapshot.html) specific indexes from a snapshot using a different name, run a `restore` API call similar to this:

    ```json
    POST /_snapshot/elasticsearch_local_backup/snapshot1/_restore
    {
      "indices": "liferay-20101-search-tuning-synonyms,liferay-20101-search-tuning-rankings",
      "ignore_unavailable": true,
      "include_global_state": false,
      "rename_pattern": "(.+)",
      "rename_replacement": "restored_$1",
      "include_aliases": false
    }
    ```

   where `indices` sets the snapshotted index names to restore from. The indexes from the above call would be restored as `restored_liferay-20101-search-tuning-rankings` and `restored_liferay-20101-search-tuning-synonyms`, following the `rename_pattern` and `rename_replacement` regular expressions.

If you've added Synonym Sets or Results Rankings while running in Sidecar/Embedded mode, you'll see these search tunings disappear once you configure a Remote mode connection to Elasticsearch 7 and perform a full reindex.

Restoring the _Search Tuning_ indexes from a snapshot using new index names is important if you've set up a new Elasticsearch 7 cluster, configured a remote connection to it, and performed a full reindex. Any 
Restoring the _Search Tuning_ indexes from a snapshot using new index names is important comes in handy if you did not upgrade your Elasticsearch 6 cluster, but rather, you just set-up your new Elasticsearch 7 cluster, configured Elasticsearch 7 in @product-ver@ and performed a full reindex from @product@.

In this case, existing _Search Tuning_ entries will be missing, because whose indexes will be empty.

To restore your existing _Search Tuning_ index documents, you can use the [Reindex API](https://www.elastic.co/guide/en/elasticsearch/reference/current/docs-reindex.html#docs-reindex) of Elasticsearch, like this:

```json
   POST _reindex/
   {
     "dest": {
       "index": "liferay-20101-search-tuning-synonyms"
     },
     "source": {
       "index": "restored_liferay-20101-search-tuning-synonyms"
     }
   }
```

```tip::
   It's convenient to create and manage snapshots via the `Kibana 7.x UI <https://www.elastic.co/guide/en/kibana/7.x/snapshot-repositories.html>`__.
```

### Search Tuning Index Names

The out-of-the-box Search Tuning index names depend on your Liferay version and patch level:

| Liferay Version and Patch | Search Tuning Indexes |
| ------------------------- | --------------------- |
| Liferay DXP 7.2 SP1 and SP2 | `liferay-search-tuning-rankings`<br />  |
| Liferay DXP 7.2 SP2 | `liferay-search-tuning-rankings`<br />`liferay-search-tuning-synonyms-liferay-<companyId>` |
| Liferay DXP 7.2 SP3+/FP8+ | `liferay-<companyId>-search-tuning-rankings`<br />`liferay-<companyId>-search-tuning-synonyms` |
| Liferay DXP 7.3, all patches  | `liferay-<companyId>-search-tuning-rankings`<br />`liferay-<companyId>-search-tuning-synonyms` |

The `<companyId>` (e.g., `20101`) belongs to a given `Company` record in the database. It is displayed as _Instance ID_ in the UI and represents a [Virtual Instance](../../../../system-administration/configuring-liferay/virtual-instances/understanding-virtual-instances.md).
