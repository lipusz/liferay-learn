# What's New in Search for 7.3

## Elasticsearch Integration

### Elasticsearch 7 Support

> Availability: Liferay CE 7.3 GA4+, Liferay DXP 7.3 GA1+

Liferay DXP 7.3 comes with out-of-the-box support for Elasticsearch 7. The minimum required version is 7.9. Refer to the [Compatibility-Matrix](https://help.liferay.com/hc/en-us/sections/360002103292-Compatibility-Matrix) for detailed support information.

```important::
   Elasticsearch 6.x is not supported on Liferay CE/DXP 7.3.
```

### The Elasticsearch 7 Connector is Based on the REST Client

> Availability: Liferay CE 7.3 GA4+, Liferay DXP 7.3 GA1+

The Elasticsearch 7 connector bundled with DXP 7.3 uses [Elastic's Java REST Client](https://www.elastic.co/guide/en/elasticsearch/client/java-rest/7.8/java-rest-overview.html) to communicate with Elasticsearch over HTTP. This communication protocol does not require Java serialization between DXP and Elasticsearch, so Elasticsearch nodes can run on a different JVM than the Liferay DXP deployment.

### X-Pack Security Features Integrated

> Availability: Liferay 7.3 CE GA4+, Liferay DXP 7.3 GA1+

The functionality of the _Liferay Connector to X-Pack Security_ module is included in the _Liferay Connector to Elasticsearch 7_. All customers on CE/DXP 7.3 can connect securely to their Elasticsearch cluster without a [Liferay Enterprise Search (LES)](https://www.liferay.com/products/dxp/enterprise-search) subscription.

The [upgrade](../installing-and-upgrading-a-search-engine/upgrading-search-to-73.md) guide will contain information on moving the encryption configuration into the Elasticsearch 7 connector.

### Multiple Elasticsearch Connections

> Availability: Liferay CE 7.3 GA4+, Liferay DXP 7.3 GA1+

Configure connections to multiple Elasticsearch clusters. In Liferay DXP 7.2 it was possible to query against any index (even non-Liferay indices) in the Elasticsearch cluster that Liferay was connected to. It's now possible to query against indices in non-Liferay Elasticsearch clusters. This feature can be used in conjunction with the Low Level Search Options and Search Results widgets to show results from third-party systems using Elasticsearch clusters. This functionality was added primarily to support [Cross-Cluster Replication](#cross-cluster-replication).

### Embedded Mode Replaced with Sidecar

> Availability: Liferay CE 7.3 GA4+, Liferay DXP 7.3 GA1+

See the [Sidecar](#a-sidecar-elasticsearch-7-is-bundled) section under [Development(#development).

## Search Infrastructure & Administration

### Multi-Tenant Index Names

> Availability: Liferay 7.3 CE GA6+, Liferay DXP 7.3 GA1+ (also in Liferay DXP 7.2 SP3+/FP8+)

TO BE WRITTEN

### Connections in Search Admin

> Availability: Liferay CE 7.3 GA1+, Liferay DXP 7.3 GA1+

View the status and health of Elasticsearch connections via the Control Panel's Search entry (found in the Configurations section). See

- the health of each connected Elasticsearch cluster
- the node names and versions within each cluster
- the Elasticsearch client versions

This replaces the information bar found on the top of the Index Actions page in prior versions.

### Standardized Index Document IDs

> Availability: Liferay CE 7.3 GA1+, Liferay DXP 7.3 GA1+

The UID (or `id`) of index documents has 1-to-1 parity with the database rows of Liferay DXP entities.

## Search Widgets

### Customize Search Widgets Using Widget Templates

> Availability: Liferay CE 7.3 GA4+, Liferay DXP 7.3 GA1+

The Search widgets now support [Widget Templates]( ./../../site-building/displaying-content/customizing-widgets/styling-widgets-with-widget-templates.md) (previously known as Application Display Templates). This allows customizing the visual look and feel of each widget using Freemarker or Velocity templates. For example, the Search Results widget can be configured to display a card layout, and an asset's properties like author or modified date can be shown or hidden. Some default templates are provided out of the box for each supported search widget.

### Similar Results (DXP)

> Availability: Bundled with Liferay DXP 7.3 GA1+, installable on Liferay DXP 7.2 SP2+ via the [Similar Results](https://web.liferay.com/marketplace/-/mp/application/172465398) Marketplace app

The Similar Results widget displays a collection of assets similar to the main asset on a page. The widget determines the main asset display using the URL and leverages Elasticsearch's [more like this query](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html) to find additional assets. The main asset can be displayed using a display widget (like the Blogs widget) or the Asset Publisher.

The first version of Similar Results supports Blogs, Message Boards, Documents, and Wikis. Support for additional asset types (both native and custom assets) is possible by implementing an [extension point](../developer-guide/writing-a-similar-results-contributor.md). The logic used to determine document similarity can be configured per Elasticsearch's More Like This [query parameters](https://www.elastic.co/guide/en/elasticsearch/reference/current/query-dsl-mlt-query.html). 

See the [Similar Results](../search-pages-and-widgets/similar-results.md) article for details.

## Search Tuning (DXP)

### Search Tuning: Result Rankings (DXP)

NOT NEW
Availability: Liferay DXP 7.2 SP1+, Liferay DXP 7.3 GA1+

Result Rankings allows search administrators to custom tune the result relevancy for a given query using a graphical UI. Result Rankings features three main capabilities to manually tune relevancy:
1. Results for a query can be pinned and ordered at the top of the list. If the desired document is not in the original result list, it can be manually added. This allows users to promote high-value results.
1.  Results can be hidden from the result list. This capability can be used to manually remove results that are stale or irrelevant.
1. Aliases apply the same custom pinned and hidden results to alternate search queries. For example, if a result ranking is created with pinned and hidden results for the query "digital experience platform", adding the aliases "portal" and "dxp" will apply the same pinned and hidden results.

**This feature requires Elasticsearch as a configured search engine for DXP**.

Documentation: LINK-TO
Internal guide: https://grow.liferay.com/learn/Search+Tuning+-+Result+Rankings+in+DXP+Support+Guide

### Search Tunings: Synonyms (DXP)

NOT NEW
Availability: Liferay DXP 7.2 SP1+, Liferay DXP 7.3 GA1+

Synonyms allow search administrators to relate queries with similar meaning. Equivalent weight will be applied to all queries in a synonym set when searching. For example, the queries "mobile phone", "cell phone", and "hand phone" have equivalent meanings and can be used interchangeably. Defining a synonym set with these three queries will allow users searching for "mobile phone" to find documents containing the words "cell phone" or "hand phone". See brief summary about "synonyms in Elasticsearch" [here](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms.html).

**This feature requires Elasticsearch as a configured search engine for DXP**.

Documentation: LINK-TO
Internal guide: https://grow.liferay.com/learn/Search+Tuning+-+Synonyms+in+DXP+Support+Guide

## Liferay Enterprise Search (LES)

The following features requires an active [Liferay Enterprise Search (LES)](https://www.liferay.com/products/dxp/enterprise-search) subscription.

### Cross-Cluster Replication (LES)

> Availability: Liferay DXP 7.2 SP3+, Liferay DXP 7.3 GA1+ via the [Liferay Enterprise Search Cross-Cluster Replication]() Marketplace application.

The Cross-Cluster Replication application replicates Elasticsearch clusters across remote data centers for disaster recovery (high availability) and geo-proximity performance optimization. It leverages Elasticsearch's [Cross-Cluster Replication](https://www.elastic.co/guide/en/elasticsearch/reference/7.9/xpack-ccr.html) feature which replicates search indexes across distributed clusters using an active-passive model. All write operations for all DXP nodes route to a single leader index, while each DXP cluster node can be configured to read from any follower index.

### Learning to Rank (LES)

NOT NEW

> Availability: Liferay DXP 7.2 SP2+, Liferay DXP 7.3 GA1+ via the [Liferay Enterprise Search Learning to Rank](https://web.liferay.com/marketplace/-/mp/application/170666298)

Manually tuning search for optimal results is difficult. Optimizing your search algorithm for one specific query may unintentionally result in poorer results for hundreds of other queries.

Learning to Rank (LTR) tackles this challenge by applying a machine learning (ML) model to improving search results.The model is trained by 

- selecting "features" (a ML term for the search factors to consider when the model is trained---e.g., recency, geo-proximity, tags match)
- providing judgment lists (also known as "ground truth") curated either manually (e.g. search experts grading results for a query) or semi-automatically (e.g. measuring user click rate for a query).

Judgment lists are the source of truth that informs and guides the model training. When a user submits a search query, LTR takes the first _x_ number of results (by default the first 1,000 results) returned by the search engine, then re-scores and re-orders those results using the trained model before returning the list to the user.

### Elasticsearch Monitoring (LES)

NOT NEW
SHOULD WE INSTEAD DISCUSS THE RENAME OF ALL LES APPS?

The monitoring app was renamed from _Connector to X-Pack Monitoring_ to _Liferay Enterprise Search Monitoring_.

## Development

### New Index Settings Contributor Extension Point

> Availability: Liferay CE 7.3 GA6+, Liferay DXP 7.3 GA1+

An Elasticsearch-version agnostic `IndexSettingsContributor` was added ([src code here](https://github.com/liferay/liferay-portal/blob/[$LIFERAY_LEARN_DXP_GIT_TAG$]/modules/apps/portal-search/portal-search-spi/src/main/java/com/liferay/portal/search/spi/settings/IndexSettingsContributor.java):

* `com.liferay.portal.search.spi.settings.IndexSettingsContributor`
* `com.liferay.portal.search.spi.settings.IndexSettingsHelper.java`
* `com.liferay.portal.search.spi.settings.TypeMappingsHelper.java`

### A Sidecar Elasticsearch 7 is Bundled

> Availability: Liferay CE 7.3 GA4+, Liferay DXP 7.3 GA1+

The Open Source (OSS) version of Elasticsearch 7.9.0 (licensed under Apache 2.0) is bundled with the Liferay CE/DXP 7.3 Tomcat bundles and started simultaneously in a _sidecar_ JVM. This sidecar installation is referred to as _development mode_, and it replaces the previous _embedded_ mode (neither of which are supported for production installations). Sidecar has dedicated properties in the Elasticsearch 7 configuration in the System Settings. By default, it runs on HTTP port 9201.See [Understanding Sidecar](../installing-and-upgrading-a-search-engine/sidecar.md) for more information.

The Elasticsearch 7.9.0 OSS version is auto-downloaded at first startup. Please refer to the article on manually installing a [sidecar server](../installing-and-upgrading-a-search-engine/manually-installing-sidecar.md) if your deployment is not able to reach external sites to download resources.

## Upgrading to DXP 7.3

Upgrading from a previous DXP version to 7.3 requires some special steps due to the switch the [REST client](#the-elasticsearch-7-connector-is-based-on-the-rest-client). Please consult with [the upgrade documentation](../installing-and-upgrading-a-search-engine/upgrading-elasticsearch.md) for important information to assist your upgrade journey.
