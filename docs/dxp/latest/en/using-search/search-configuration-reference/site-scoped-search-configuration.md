# Site Scoped Search Configuration

By the strict definition of [Site Scoped Configuration](https://help.liferay.com/hc/en-us/articles/360029131551-Introduction-to-Setting-Up#configuration-scope), search doesn't have any. However, [Search Pages](https://help.liferay.com/hc/en-us/articles/360028821052-Configuring-Search-Pages) influence Site-specific search behavior. Commonly, Search Pages contain search widgets configured to search for all content within a particular Site.

![Configure the scope of a search.](./configuring-search/images/02.png)

There are some important configuration nuances to be aware of when using the Search widgets:

If the Header Search (the search bar embedded in the default theme) uses the Search Bar widget, its configuration always requires a _destination page_ to be set, where Users are redirected to complete their search activity, interacting with the other Search widgets (Results, Facets, Suggestions etc.). [Search destination pages](https://help.liferay.com/hc/en-us/articles/360028821052-Configuring-Search-Pages) are ordinary pages holding the Search widgets. You can have as many pages with Search widgets across the Site as you want.

The Search Bar widget is instanceable, so one page can contain multiple Search Bar widgets configured differently. All Search Bar instances must point to a Search Page within the Site to be effective.

```important::
    If the destination Search Page has a Search Bar widget instance besides the embedded Header Search, the configurations of the Header Search take precedence over the page's widget instance.

    Conversely, searching from a Search Bar widget instance on other pages honors their configurations, even if they differ from the Header Search configuration.
```

See the documentation on [configuring a Search Bar](https://help.liferay.com/hc/en-us/articles/360029133811-Searching-for-Assets#configuring-the-search-bar) for more information.
