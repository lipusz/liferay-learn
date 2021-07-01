# Search Configuration Reference

Search in Liferay DXP can be configured in lots of ways. Once you understand the concept of [configuration scope](../system-administration/configuring-liferay/understanding-configuration-scope.md), you'll understand where to find a certain configuration screen.

* Many search configurations affect the system scope (e.g., re-indexing actions and configuring the search engine connector).
* Search tuning configurations affect the virtual instance.
* Creating new Search Pages affects search at the site scope.
* Configuring the search widgets is widget-scoped configuration, but some can also be thought of as page-scoped configuration.

So the term `Configuring Search` is very broad, as it addresses all the available scopes. This is a high level overview of what search behavior is configurable out of the box, and importantly, where to find search configuration options.

## Widget Scoped Search Configuration

Several search widgets are available to place on Pages. Each one has its own configuration options.

![Each widget's configuration will be unique.](./search-configuration-reference/images/03.png)

### Search Widgets

For dedicated coverage of the search widgets, see [Search Pages and Widgets](../getting-started/search-overview.md#search-pages-and-widgets).

**Search Results**: Configure how search results are displayed. Read [Displaying Search Results](../search-pages-and-widgets/search-results/configuring-the-search-results-widget.md#displaying-search-results) for more information.

**Search Bar**: Configure the behavior of how search keywords are processed. See [Configuring the Search Bar](../getting-started/searching-for-content.md#configuring-the-search-bar) for more information.

**Search Facets**: Configure each facet's behavior and URL parameters. See [Facets](../search-pages-and-widgets/search-facets/facets.md) for more information.

**Search Options**: This is a special case, where configuring this widget defines page scoped behavior. Add the Search Option widget to a page and define two booleans for the Search Page:

* Allow Empty Searches: By default, failure to enter a keyword returns no results. Enabling this ensure that _all_ results are returned when no keyword is entered in the Search Bar.
* Basic Facet Selection: By default, facet counts are recalculated after each facet selection. Enable this to turn off facet recounting.

**Search Suggestions**: Suggest better queries and spell check queries. See the [Suggestions Configuration Reference](../search-pages-and-widgets/search-results/enabling-search-suggestions.md#suggestions-configuration-reference) for more information.

**Search Insights**: Add this to the Search Page to inspect the full query string that's constructed by the back-end search code when the User enters a keyword. Only useful for testing and development. See [Search Insights](../search-pages-and-widgets/search-insights.md) for more information.

**Custom Filter**: Add a widget to the page for each of the filters you'd like applied to the search results. Let search page users see and manipulate the filters or make them invisible and/or immutable. See [Filtering Search Results](../search-pages-and-widgets/search-results/filtering-search-results.md) for more information.

**Sort**: Let Users reorder the search results based on the value of certain `keyword` fields in the index. For example, show results in alphabetic order of the Title field. The default order is determined by the search engine's _Relevance_ calculation. See [Sorting Search Results](../search-pages-and-widgets/search-results/sorting-search-results.md) for more information.

**Low Level Search Options:** Configure the search widgets to participate in a search that's aimed at an index other than the _Company Index_. The company index is where the Liferay DXP assets index their data, so many installations will not need this widget. See [Understanding Low Level Search Options](../search-pages-and-widgets/search-results/understanding-low-level-search-options.md) for more information.

**Similar Results:** Display similar search results to an asset being displayed on a page. See [Similar Results](../search-pages-and-widgets/similar-results.md) for more information.

[Liferay Enterprise Search] **Elasticsearch Monitoring:** Liferay Enterprise Search subscribers can access [Elastic's Kibana](https://www.elastic.co/kibana) monitoring tool inside a widget placed on a Liferay DXP Page. See [Monitoring Elasticsearch](../liferay-enterprise-search/monitoring-elasticsearch.md) for more information.

## Site Scoped Search Configuration

By the strict definition of [Site Scoped Configuration](../../system-administration/configuring-liferay/understanding-configuration-scope.md), search doesn't have any. However, [Search Pages](../search-pages-and-widgets/working-with-search-pages/search-pages.md) influence Site-specific search behavior. Commonly, Search Pages contain search widgets configured to search for all content within a particular Site.

![Configure the scope of a search.](./search-configuration-reference/images/04.png)

There are some important configuration nuances to be aware of when using the Search widgets:

If the header Search Bar (the search bar embedded in the default theme) uses the Search Bar widget, its configuration always requires a _destination page_ to be set, where Users are redirected to complete their search activity, interacting with the other Search widgets (Results, Facets, Suggestions etc.). [Search destination pages](../search-pages-and-widgets/working-with-search-pages/creating-a-search-page.md) are ordinary pages holding the Search widgets. You can have as many pages with Search widgets across the Site as you want.

The Search Bar widget is instanceable, so one page can contain multiple Search Bar widgets configured differently. All Search Bar instances must point to a Search Page within the Site to be effective.

```important::
    If the destination Search Page has a Search Bar widget instance in addition to a theme-embedded header Search Bar (like Liferay's default theme), the configurations of the header Search Bar take precedence over the page's widget instance.

    Conversely, searching from a Search Bar widget instance on other pages honors their configurations, even if they differ from the header Search Bar configuration.
```

See [Configuring the Search Bar](../getting-started/searching-for-content.md#configuring-the-search-bar) for more information.

## Instance Scoped Search Configuration

Search does not have any Instance scoped entries in the [Instance Settings](../../system-administration/configuring-liferay/understanding-configuration-scope.md#system-settings-and-instance-settings) panel in Liferay 7.3 and below. However, there are entries for configuring the instance-wide default [widget template](../../site-building/displaying-content/additional-content-display-options/styling-widgets-with-widget-templates.md) for many search widgets. Just enter the Site ID where the template is defined (often this will be the Global site's ID, found in its Site Settings panel) and the Widget Template ID, found in the Site Menu &rarr; Design &rarr; Widget Templates.

The only instance scoped configurations for search are the entries in the Global Menu &rarr; Applications &rarr; Search Tuning:

**Result Rankings:** Customize search results manually by hiding, pinning, and adding results for specific aliases. See [Result Rankings](../search-administration-and-tuning/result-rankings.md) for more information.

**Synonyms:** Create synonym sets so that synonymous search terms are matched and scored like an exact match to the search term. See [Synonym Sets](../search-administration-and-tuning/synonym-sets.md) for more information.

## System Scoped Search Configuration

System scoped search configurations are primarily found in [System Settings](../../system-administration/configuring-liferay/system-settings.md).

1. In the Global Menu, go to *Control Panel* &rarr; *Configuration* &rarr; *System Settings*.

1. Click the *Search* category under the Platform section.

    Alternatively, search for *search*.

![There are numerous system scoped entries for search in System Settings.](./search-configuration-reference/images/01.png)

### Default Keyword Query

The Default Keyword Query entry contains one setting:

**`disabledEntryClassNames`**: The `DefaultKeywordQueryContributor` code automatically adds `description`, `userName`, and `title` fields to the keyword search query. Specify the entry class names `DefaultKeywordQueryContributor` should ignore.

### Default Search Result Permission Filter

The Default Search Result Permission Filter entry allows configuration of *post-filtering permission checking* (database permission checking that occurs after the results are returned from the search index). Read [Final Permissions Checking](../search-pages-and-widgets/search-results/search-results-behavior.md#final-permissions-checking) for more information on these settings:

* **`permissionFilteredSearchResultAccurateCountThreshold`**

* **`searchQueryResultWindowLimit`**

### Index Status Manager

The Index Status Manager entry has one setting:

**`indexReadOnly`**: Enable this to suspend all indexing operations and writes to the search engine. Searches return only the documents already indexed. This is useful for speeding up large data imports, but it should be disabled and a full re-index executed once the import is finished.

### Indexer Writer Helper

The Index Writer Helper entry contains one entry:

**`indexCommitImmediately`**: When *true* (the default), each write request forces the search engine to refresh the index reader, potentially flushing transactions to disk. This may negatively impact search engine performance. The default behavior is to commit immediately for index writing on individual assets (e.g. add blog, update blog) but delay commits for bulk index writing operations (e.g.  index all users, index all form entries) until all entries have been sent to the search engine. Setting this to false changes the behavior for individual index operations, and may cause applications like Asset Publisher to exhibit a delayed response when showing newly added content. See the [Elasticsearch documentation](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/near-real-time.html) for more information.

### Index Registry

Configure the buffering of index requests:

**`buffered`**: Enable (the default) or disable the buffering of indexing requests.

**`bufferedExecutionMode`**: Allow administrators to select a different `IndexerRequestBufferExecutor`, used to execute a `IndexerRequest`. One implementation of the executor is provided out of the box (_DEFAULT_). When a new `IndexerRequestBufferExecutor` implementation is deployed, one of the required properties is a `buffered.execution.mode`. This value can then be used in place of _DEFAULT_.

**`maximumBufferSize`**: If buffering is enabled, set the Maximum Buffer Size so that any additional indexing requests are executed immediately.

**`minimumBufferAvailabilityPercentage`**: When the capacity of the buffer has only the specified percent of space left, the existing requests in the buffer are executed in one batch and removed from the buffer.

### Index Query Preprocessor

This entry has one repeatable property (use array syntax if you're defining via [OSGi configuration file](../../system-administration/configuring-liferay/configuration-files-and-factories/using-configuration-files.md#creating-configuration-files)):

**`fieldNamePatterns`**: Fields with names matching the patterns set here are treated as non-analyzed keyword fields. Instead of scored full text queries, matching is performed by non-scored wildcard queries. This is a resource intensive operation that degrades search engine performance as indexes grow larger. For substring matching, relying on the [ngram tokenizer](https://www.elastic.co/guide/en/elasticsearch/reference/current/analysis-ngram-tokenizer.html) usually performs better.

### Reindex

This entry contains only one property:

**`indexingBatchSizes`**: Set the number of documents (the default value is 1000) indexed per batch for model types that support batch indexing. For models with large documents, decreasing this value may improve stability when executing a full re-index.

### Reindexer

These properties configure the Reindexer service in the Search framework. They are experimental and only useful when troubleshooting certain specific scenarios under the direction of Liferay's support personnel. Never use these configurations in production systems. 

**`nonbulkIndexingOverride`**: Set this to true to force modified entities to be re-indexed one by one rather than in batches. For performance reasons, this setting must not be enabled in production systems.

**`synchronousExecutionOverride`**: Set this to true to force re-indexing of modified entities to block other portal processes until completed. For performance reasons, this setting must not be enabled in production systems.

### Engine Helper

This entry has one repeatable property (use array syntax if you're defining via
[OSGi configuration file](../../system-administration/configuring-liferay/configuration-files-and-factories/using-configuration-files.md)):

**`excludedEntryClassNames`**: Exclude an asset type from being searched in the catchall query constructed for the Search application. For example, fields of the Organization asset must be indexed to be searchable from the Users and Organizations application, but should not be searched in the Search application. Thus, Organizations are added to `excludedEntryClassNames`.

### Permission Checker

This entry has one configuration option in Liferay DXP 7.3:

**`permissionTermsLimit`**: Limits the number of permission search clauses added to the search query before this level of permission checking is aborted. Permission checking then relies solely on the final permission filtering described in the [Default Search Result Permission Filter](#default-search-result-permission-filter) section.

### Title Field Query Builder

Configure how search responds to matches on the Title field of a document.

**`exactMatchBoost`**: Give an additional boost when searched keywords exactly match the `title` field of a document.

**`maxExpansions`**: Limit the number of documents to return when matching searched keywords to the `title` field as a phrase prefix. See Elasticsearch's [Match Phrase Query documentation](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-match-query-phrase.html) for more information.

### Cross-Cluster Replication

You'll only see this entry if you have deployed the Cross-Cluster Replication module, which is available with a Liferay Enterprise Search subscription. These configuration properties are covered in the dedicated [Cross-Cluster Replication](../liferay-enterprise-search/cross-cluster-replication/cross-cluster-replication.md) documentation.

### Elasticsearch Monitoring

You can use the Elasticsearch Monitoring configuration if you have deployed the Liferay Enterprise Search Monitoring module, available with a Liferay Enterprise Search subscription. These configuration properties are covered in the dedicated [Monitoring Elasticsearch](../liferay-enterprise-search/monitoring-elasticsearch.md) documentation.

### Elasticsearch 7

Configure the connection between Liferay and Elasticsearch 7. See [Configuring the Connector](../installing-and-upgrading-a-search-engine/elasticsearch/connecting-to-elasticsearch.md#configuring-the-connector) for detailed coverage of these properties.

### Elasticsearch Connections

Liferay 7.3 can connect to multiple Elasticsearch clusters. Connections are defined in the Elasticsearch Connections entry. This feature can be used in conjunction with the Low Level Search Options and Search Results widgets to show results from third-party systems using Elasticsearch clusters. It's also demonstrated in the [Cross-Cluster Replication](../liferay-enterprise-search/cross-cluster-replication/cross-cluster-replication.md) documentation.

### Learning to Rank

You can use the Learning to Rank configuration if you have deployed the Liferay Enterprise Search Learning to Rank module, available with a Liferay Enterprise Search subscription. These configuration properties are covered in the dedicated [Learning to Rank](../liferay-enterprise-search/learning-to-rank.md) documentation.

### Search Web

This entry contains one property:

**`classicSearchPortletInFrontPage`**: Revert the default search experience from using the new Search Widgets to the deprecated classic Search Portlet that was standard in past releases.

### Search Administration

In *Control Panel* &rarr; *Configuration* &rarr; *Search* there are three administrative UIs: Connections, Index Actions, and Field Mappings.

![The Search Administration panel contains three management screens.](./search-configuration-reference/images/02.png)

#### Connections

Information about the search engine connection(s) is displayed here. For example,

```bash
Search Engine Vendor: Elasticsearch (Embedded), Client Version: 6.5.0, Nodes: A0D6GlN (6.5.0)
```


#### Index Actions

In Index Actions, re-index at one of these levels:

   * All indexable assets
   * An individual indexable asset
   * All spell check indexes

#### Field Mappings

The Field Mappings tab shows you all field mappings that are effective in the system, by index. Currently, you can view the mappings, copy them, zoom in or out, and view them with a dark theme.

### Portal Properties

Portal properties are system scoped as well. The [Lucene Search](https://docs.liferay.com/dxp/portal/7.3-latest/propertiesdoc/portal.properties.html#Lucene%20Search) portal properties configure low level search behavior. Review the properties and their descriptions and determine if they apply to your search requirements. -->
