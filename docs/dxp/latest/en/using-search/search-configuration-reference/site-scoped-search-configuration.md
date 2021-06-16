# Site Scoped Search Configuration

By the strict definition of [Site Scoped Configuration](../../system-administration/configuring-liferay/understanding-configuration-scope.md), search doesn't have any. However, [Search Pages](../search-pages-and-widgets/working-with-search-pages/search-pages.md) influence Site-specific search behavior. Commonly, Search Pages contain search widgets configured to search for all content within a particular Site.

![Configure the scope of a search.](./configuring-search/images/02.png)

There are some important configuration nuances to be aware of when using the Search widgets:

If the Header Search (the search bar embedded in the default theme) uses the Search Bar widget, its configuration always requires a _destination page_ to be set, where Users are redirected to complete their search activity, interacting with the other Search widgets (Results, Facets, Suggestions etc.). [Search destination pages](../search-pages-and-widgets/working-with-search-pages/creating-a-search-page.md) are ordinary pages holding the Search widgets. You can have as many pages with Search widgets across the Site as you want.

The Search Bar widget is instanceable, so one page can contain multiple Search Bar widgets configured differently. All Search Bar instances must point to a Search Page within the Site to be effective.

```important::
    If the destination Search Page has a Search Bar widget instance besides the embedded Header Search, the configurations of the Header Search take precedence over the page's widget instance.

    Conversely, searching from a Search Bar widget instance on other pages honors their configurations, even if they differ from the Header Search configuration.
```

See [Configuring the Search Bar](../getting-started/searching-for-content.md#configuring-the-search-bar) for more information.
