# Upgrading to Elasticsearch 7.9

Elasticsearch 7.9+ is supported for Liferay 7.3 out of the box. For Liferay 7.2 it's supported via the [Liferay Connector to Elasticsearch 7](https://web.liferay.com/marketplace/-/mp/application/170390307) (version `3.x`). If you're upgrading to Liferay 7.3 or wanting to bring your existing 7.2 search engine to Elasticsearch 7.9+, you must upgrade your Elasticsearch servers. If you're setting up a new Liferay 7.2 system, install Elasticsearch 7.9 and follow the [installation guide](../getting-started-with-elasticsearch.md).

```important::
   Before upgrading Elasticsearch, back up your existing data. If something goes wrong during or after the upgrade, roll back to the previous version using the uncorrupted index snapshots. Follow the steps in `Backing up Elasticsearch <./backing-up-elasticsearch.md>`__.
```

To upgrade an existing Elasticsearch server (or cluster) to Elasticsearch 7.9+,

1. [Back up the Liferay Company and System Indexes](./backing-up-elasticsearch.md).

1. [Back up the application specific indexes for Search Tuning](./backing-up-elasticsearch.md#backing-up-and-restoring-search-tuning-indexes) (Synonym Sets and Result Rankings).

1. [Install and configure Elasticsearch 7.9](../installing-elasticsearch.md).

1. [Upgrade Elasticsearch](#upgrading-elasticsearch)

1. If you're using X-Pack security, make sure it's enabled:

   ```yaml
   xpack.security.enabled: true
   ```

   See [Securing Elasticsearch](../securing-elasticsearch.md) for detailed coverage of the security configuration.

1. \[7.2 only\] [Blacklist the bundled Liferay Connector to Elasticsearch 6](#blacklisting-elasticsearch-6) and [install](../connecting-to-elasticsearch.md#install-the-elasticsearch-7-connector) the Connector to Elasticsearch 7.

1. Configure the Connector to Elasticsearch 7 to connect to Elasticsearch 7.9+.

1. Re-index all search and spell check indexes.

1. Verify that Search Tuning entries have been carried over.

## Upgrading Elasticsearch

<!-- does this make sense here? Does it apply when going from 6.x to 7.9+ -->
If you are using a rolling-restart eligible version (`6.8.x`), doing a [rolling upgrade](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/rolling-upgrades.html) is the recommended way to ugprade your Elasticsearch cluster. Otherwise, follow the [full cluster restart upgrade
](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/restart-upgrade.html) guide.

## Blacklisting Elasticsearch 6

To blacklist the bundled Elasticsearch 6 connector on 7.2,

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

## Re-index

Once Liferay is connected with the Elasticsearch cluster, re-index the applicable indexes into the new Elasticsearch installation:

1. Re-index the company, system, and spell check indexes: from the Global Menu (![Global Menu](../../../../images/icon-applications-menu.png)),navigate to *Control Panel* &rarr; *Configuration* &rarr; *Search*. Click *Execute* for the *Reindex all search indexes* entry.

1. Re-index the [Workflow Metrics indexes](../../../../process-automation/workflow/user-guide/workflow-metrics-reports.html#re-indexing-workflow-metrics): from the Global Menu (![Global Menu](../../../../images/icon-applications-menu.png)), navigate to *Applications* &rarr; *Workflow---Metrics*. Open the Settings menu (![Options](../../../../images/icon-options.png) and Click *Reindex All*.

This restores the indexes that are built from data stored in the Liferay databse. To restore indexes that are used as primary storage, see [Backing Up Elasticsearch](./backing-up-elasticsearch.md).

## Liferay 7.2: Reverting to Elasticsearch 6
<!-- Is this still useful to maintain? Would it be better, now that we could have people upgrading from Elasticsearch 7.3-7.9, to genericize these steps to be version agnostic? -->

Stuff happens. If you're on Liferay 7.2 and that stuff involves an unrecoverable failure during the upgrade to Elasticsearch 7, roll back to Elasticsearch 6 and regroup.

Since your Elasticsearch 6 and 7 are currently two separate installations, this procedure takes only a few steps:

1.  Stop the Liferay Connector to Elasticsearch 7.

1.  Stop Elasticsearch 7 and make sure that the Elasticsearch 6 `elasticsearch.yml` and the connector app are configured to use the same port (9200 by default).

1.  Start the Elasticsearch server, and then restart the Liferay Connector to Elasticsearch 6.

Once your upgrade is completed, see the [new search feautres available in Liferay 7.3](../../../getting-started/whats-new-in-search-for-73.md). 
