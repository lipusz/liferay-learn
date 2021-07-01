# Widget Scoped Search Configuration

Several search widgets are available to place on Pages. Each one has its own configuration options.

![Each widget's configuration will be unique.](./widget-scoped-search-configuration/images/01.png)

## Search Widgets

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
