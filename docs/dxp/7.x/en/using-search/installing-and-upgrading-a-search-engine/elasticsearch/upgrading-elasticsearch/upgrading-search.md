# Upgrading Search

Liferay 7.3 brings [new improvements](../getting-started/whats-new-in-search-for-73.md) for search. You must upgrade to them. Elasticsearch 7.9 is the minimum version required for Liferay 7.3. The [compatibility matrix](https://help.liferay.com/hc/en-us/sections/360002103292-Compatibility-Matrix) provides the latest support details.

```important::
   Solr integration is now deprecated as of Liferay 7.3. Elasticsearch integration replaces it. Migrating to Elasticsearch requires `setting up Elasticsearch `<./elasticsearch/getting-started-with-elasticsearch.md>`_ and `connecting Liferay <./elasticsearch/connecting-to-elasticsearch.md`_ to it.
```

```important::
   Elasticsearch 6.x is not supported on Liferay 7.3.
```

Here are the general upgrade steps:

1. Back up your search indexes.
1. Set up the latest supported search engine.
1. Set up Liferay's connector to that search engine.
1. Configure security.
1. Upgrade the Liferay database.
1. Re-index search indexes.

Examine [Upgrading Search to 7.3 Overview](./upgrading-search-for-liferay-73.md) first because it summarizes upgrading from search stacks on previous Liferay versions to the latest supported search stack. Then [back up your search indexes]((./elasticsearch/backing-up-elasticsearch.md)) before [upgrading search](./elasticsearch/upgrading-elasticsearch.md).

## Additional information

* [Backing Up Elasticsearch](./elasticsearch/backing-up-elasticsearch.md)
* [Upgrading Elasticsearch](./elasticsearch/upgrading-elasticsearch.md)
