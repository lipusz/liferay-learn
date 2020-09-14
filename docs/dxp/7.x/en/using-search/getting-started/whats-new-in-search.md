# What's New in Search

## Elasticsearch Integration

### Elasticsearch 7 Support
Availability: `7.3 CE GA4+, DXP 7.3 GA1+`

Liferay DXP 7.3 comes with out-of-the-box support for Elasticsearch 7, no need to install the Elasticsearch 7 connector an an extra step. The minimum required version is 7.9. Please refer to the [Compatibility-Matrix](https://help.liferay.com/hc/en-us/sections/360002103292-Compatibility-Matrix) which always contains information about the latest supported major and minor versions of Elasticsearch.

Elasticsearch 6.x is not supported on DXP 7.3.

### REST Client based Elasticsearch 7 Connector
Availability: `7.3 CE GA4+, DXP 7.3 GA1+`

The Elasticsearch 7 connector that is be bundled with DXP 7.3 by default uses the [Java REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.8/java-rest-overview.html) which communicates with Elasticsearch over HTTP. This communication protocol does not require Java serialization between DXP and Elasticsearch as opposed to the previous Transport Client-based connectors. Your Elasticsearch nodes can run on different JVM than what your DXP deployment is using.

### X-Pack Security Features Integrated
Availability: `7.3 CE GA4+, DXP 7.3 GA1+`

The functionality of the _Liferay Connector to X-Pack Security_ module is included with _Liferay Connector to Elasticsearch 7_ meaning all customers on DXP 7.3 can connect securely to their Elasticsearch cluster even without an [Liferay Enterprise Search (LES)](https://www.liferay.com/products/dxp/enterprise-search) subscription.

Please consult with the LINK-TO Upgrading to DXP 7.3 - Search guide if you were previously using encrypted communication (TLS/SSL) and/or authentication in your DXP-Elasticsearch stack.

### Multiple Elasticsearch Connections
Availability: `7.3 CE GA4+, DXP 7.3 GA1+`

Search admins can configure connections to multiple Elasticsearch clusters. In DXP 7.2, it was possible to query against any index (even non-Liferay indices) in the same Elasticsearch cluster that Liferay was connected to. It's now possible to query against indices in non-Liferay Elasticsearch clusters. This feature can be used in conjunction with the Low Level Search Options and Search Results widgets to show results from third-party systems using Elasticsearch clusters. Note, this feature was developed primarily to support Cross-Cluster Replication (see below).


### Embedded Mode Replaced with Sidecar
Availability: `7.3 CE GA4+, DXP 7.3 GA1+`

See the _Sidecar Elasticsearch 7_ section under the development related changes.

## Search Infrastructure & Administration

### Multi-Tenant Index Names
Availability: `7.3 CE GA6+, DXP 7.3 GA1+` (and also in `DXP 7.2 SP3+/FP8+`, see [here](https://grow.liferay.com/share/Important+Changes+in+DXP+7.2+SP3_SLASH_FP8+for+Search+Tunings+and+Workflow+Metrics+-+Multi-Tenant+Elasticsearch+Index+Names+Support))
TBW (with Elasticsearch only)

### Connections in Search Admin
Availability: `7.3 CE GA1+, DXP 7.3 GA1+`

Administrators can now view the status and health of their Elasticsearch connections via Control Panel's Search portlet. This portlet will display the health of each connected Elasticsearch cluster, the node names and versions within each cluster, and the Elasticsearch client versions. This replaces the information bar found on the top of the Index Actions page in prior versions.

### Standardized Index Document IDs
Availability: `7.3 CE GA1+, DXP 7.3 GA1+`

Developers and administrators can expect the UID (or `id`) of index documents to have 1-to-1 parity with database rows of Liferay DXP entities.

## Search Widgets

### Customizing Search Widgets using ADTs
Availability: `7.3 CE GA2?+`, `DXP 7.3 GA1+`

Search widgets now support Widget Templates (previously known as "Application Display Templates"). This allows customizing the search widgets’ visual look and feel using Freemarker or Velocity templates. For example, the Search Results widget can be configured to display a card layout, and an asset’s properties like author or modified date can be shown or hidden. A couple default templates are provided OOTB for each supported search widget.

### Similar Results (DXP)
Availability: `DXP 7.2 SP2+`via the [Similar Results](https://web.liferay.com/marketplace/-/mp/application/172465398) Marketplace app, `DXP 7.3 GA1+` (bundled)

This widget displays a collection of assets similar to the main asset on a page. The widget determines the main asset display using the URL and leverages Elasticsearch's [more like this query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html) to find more assets with similar content. The main asset can be displayed using a display widget (like the Blogs widget) or the Asset Publisher.

The first version of Similar Results supports Blogs, Message Boards, Documents, and Wikis. Support for additional asset types (both native and custom assets) is possible by implementing an [extension point](https://learn.liferay.com/dxp-7.x/using-search/developer-guide/writing-a-similar-results-contributor.html). The logic used to determine document similarity can be configured per Elasticsearch's More Like This [query parameters](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html). The "main" asset on a page is determined by the URL of the page. The widget ships with three display layouts (compact, list, and card) and supports customization through widget templates (ADT's).

Documentation: https://learn.liferay.com/dxp/7.x/en/using-search/search-pages-and-widgets/similar-results.html
Internal guide: https://grow.liferay.com/learn/Similar+Results+in+DXP+Support+Guide

## Search Tunings (DXP)

### Search Tunings: Result Rankings (DXP)
Availability: `DXP 7.2 SP1+`, `DXP 7.3 GA1+`

Result Rankings allows search administrators to custom tune the result relevancy for a given query using a graphical UI. Result Rankings features three main capabilities to manually tune relevancy:
1. Results for a query can be pinned and ordered at the top of the list. If the desired document is not in the original result list, it can be manually added. This allows users to promote high-value results.
1.  Results can be hidden from the result list. This capability can be used to manually remove results that are stale or irrelevant.
1. Aliases apply the same custom pinned and hidden results to alternate search queries. For example, if a result ranking is created with pinned and hidden results for the query "digital experience platform", adding the aliases "portal" and "dxp" will apply the same pinned and hidden results.

**This feature requires Elasticsearch as a configured search engine for DXP**.

Documentation: LINK-TO
Internal guide: https://grow.liferay.com/learn/Search+Tuning+-+Result+Rankings+in+DXP+Support+Guide

### Search Tunings: Synonyms (DXP)
Availability: `DXP 7.2 SP1+`, `DXP 7.3 GA1+`

Synonyms allow search administrators to relate queries with similar meaning. Equivalent weight will be applied to all queries in a synonym set when searching. For example, the queries "mobile phone", "cell phone", and "hand phone" have equivalent meanings and can be used interchangeably. Defining a synonym set with these three queries will allow users searching for "mobile phone" to find documents containing the words "cell phone" or "hand phone". See brief summary about "synonyms in Elasticsearch" [here](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms.html).

**This feature requires Elasticsearch as a configured search engine for DXP**.

Documentation: LINK-TO
Internal guide: https://grow.liferay.com/learn/Search+Tuning+-+Synonyms+in+DXP+Support+Guide

## Liferay Enterprise Search (LES)

The following features requires an active [Liferay Enterprise Search (LES)](https://www.liferay.com/products/dxp/enterprise-search) subscription.

### Cross-Cluster Replication (LES)
Availability: `DXP 7.2 SP3+`, `DXP 7.3 GA1+` via the [Connector to Elasticsearch Cross Cluster Replication]

The Cross-Cluster Replication module replicates Elasticsearch clusters across remote data centers for disaster recovery (high availability) and geo-proximity performance optimization. It leverages Elasticsearch's [Cross-Cluster Replication](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/xpack-ccr.html) feature which replicates search indexes across distributed clusters using an active-passive model. This module will route all write operations for all DXP nodes to a single leader index, while each DXP cluster node can be configured to read from any follower index.

Documentation: LINK-TO
Internal guide: https://grow.liferay.com/learn/LES+Cross+Cluster+Replication+CCR+Support+Guide

### Learning to Rank (LES)
Availability: `DXP 7.2 SP2+`, `DXP 7.3 GA1+` via the [Liferay Connector to Elasticsearch Learning to Rank](https://web.liferay.com/marketplace/-/mp/application/170666298)

Tuning search for optimal results is difficult. Optimizing your search algorithm for one specific query may unintentionally result in poorer results for hundreds of other queries.

Learning to Rank (LTR) tackles this challenge by applying a machine learning model to improve search results. The model is trained by selecting "features" (a ML term for the search factors that should be considered when the model is trained like recency, geo-proximity, tags match, etc) and providing judgment lists (also known as "ground truth") curated either manually (e.g. search experts grading results for a query) or semi-automatically (e.g. measuring user click rate for a query). The judgment lists are the "north star" that informs and guides the training to develop an optimal search algorithm for all queries. When a user submits a search query, LTR will take the first x number of results (default: first 1,000 results) returned by the search engine, then rescore and re-order those results according to the trained model before returning the list to the user. (Note, only the first x number of results are rescored due to performance restraints. Rescoring all results is prohibitively expensive.)

The initial release of Liferay's Learning to Rank module will allow businesses to enable a system-wide toggle to rescore search queries using LTR. However, the process of submitting judgment lists, selecting "features", and monitoring the performance of their trained model will require technical expertise as the interface is intended for search engineers and data scientists. LTR is not a silver bullet where simply enabling the feature will automagically result in better search results. Future versions of Liferay's LTR module will provide more streamlined UI's to make curation and feature selection easier.

Documentation: 
Internal guide: https://grow.liferay.com/learn/Learning+to+Rank+LTR+-+Liferay+Enterprise+Search+Support+Guide

### Elasticsearch Monitoring (LES)
Availability: 

TBW
Renamed: previously was called as Connector to X-Pack Monitoring.

Documentation: LINK-TO
Internal guide: -

## Development

### New Extension Points & APIs
TBW

* Elasticsearch-version agnostic `IndexSettingsContributor` (https://issues.liferay.com/browse/LPS-119024)
  * `com.liferay.portal.search.spi.settings.IndexSettingsContributor`
  * `com.liferay.portal.search.spi.settings.IndexSettingsHelper.java`
  * `com.liferay.portal.search.spi.settingsTypeMappingsHelper.java`

### Sidecar Elasticsearch 7
Availability: `7.3 CE GA4+`, `DXP 7.3 GA1+`

The Open Source (OSS) version of Elasticsearch 7.9.0 (licensed under Apache 2.0) is bundled with the Tomcat bundles and started in a "sidecar" JVM when the product is running in development mode replacing the previous `Embedded` mode. This mode is now referred as `Sidecar` and it has dedicated properties in the Elasticsearch 7 configuration in the System Settings. By default, it runs on `HTTP 9201`. Please refer to the LINK-TO article for more information.

The Elasticsearch 7.9.0 OSS version is auto-downloaded at startup if  the `elasticsearch7` installation directory does not exist under your `[Liferay Home]`. Please refer to the LINK-TO article if your deployment is running under strict networking conditions and Liferay DXP is not able to reach external sites to download resources.

**Limitations**
- The [OSS version of Elasticsearch 7](https://www.elastic.co/subscriptions) dos not contain the X-Pack modules therefore you can't configure encrypted communication (TLS/SSL) or user authentication (features of X-Pack Security) using the Sidecar installation.

- Also bear in mind that only production mode deployments are supported and included in Liferay Support's coverage, sidecar is only for testing purposes and local development and it's not intended to be clustered. You must setup a standalone Elasticsearch 7.9+ deployment and configure the Elasticsearch 7 connector of Liferay DXP in `Remote` mode or set the new property called `productionModeEnabled` to `true`.

Documentation: LINK-TO
Internal guide: LINK-TO

## Upgrading to DXP 7.3

If you are upgrading from a previous DXP version, please consult with to this article that contains important information to assist your upgrade journey: https://grow.liferay.com/learn/Upgrading+to+DXP+7.3+-+Search+Elasticsearch+and+Enterprise+Search.
