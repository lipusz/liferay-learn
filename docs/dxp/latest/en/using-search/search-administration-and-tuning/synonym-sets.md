# Synonym Sets

> **Subscribers**

A Synonym Set is a group of words or phrases with the same meaning. An administrative User creates the Synonym Set; when a search page end user searches for a keyword or phrase, the synonymous terms in the set are also searched. 

For example, a user might search for the word "US". Most likely, the user would want search results that also included synonyms such as *America*, *U.S.A*, *United States*, etc. By creating Synonym Sets, you can ensure your users get the most out of their searches.

## Requirements and Limitations

Synonym Sets are only supported when using Elasticsearch as your search engine. Learn about installing Elasticsearch in [Getting Started with Elasticsearch](../installing-and-upgrading-a-search-engine/elasticsearch/getting-started-with-elasticsearch.md).

Synonym Sets currently work with fields indexed in either of the following out-of-the-box locales: English or Spanish. Liferay assets with localizable fields in either of these two languages work with Synonym Sets.

The [`=>` format](https://www.elastic.co/guide/en/elasticsearch/guide/current/synonyms.html) supported in Elasticsearch is not supported through the Synonyms Set UI.

## Creating and Managing Synonym Sets

Create a synonym set by adding as many synonymous keywords to a set as you like. Once the synonym set is saved, searches in the same company scope (any site from the [Virtual Instance](../../system-administration/configuring-liferay/virtual-instances/understanding-virtual-instances.md) where the synonyms were configured) take effect.

To create a new synonym set,

1. From the Global Menu (![Global Menu](../../images/icon-applications-menu.png)) navigate to *Applications* &rarr; *Search Tuning* &rarr; *Synonyms*.

   ![Navigate to the Synonyms section in the Applications menu](synonym-sets/images/01.png)

1. Click the _Add_ icon (![Click on the add icon](../../images/icon-add.png)) to add a new Synonym Set.

1. Enter the list of synonyms in the set. The input of a synonym is accomplished by clicking *Enter* or by entering a comma.

   ![Type your different synonym words into the set.](synonym-sets/images/02.png)

1. You can delete synonym by clicking on the *X* next to it. When you finish your set, click *Publish*.

1. To edit or delete a set, click the _Options_ icon (![Click on the options icon.](../../images/icon-options.png)) and click on *Edit* or *Delete*.

   ![Click on edit or delete to make changes.](synonym-sets/images/03.png)

   Once your synonym set is published, it is ready to be used.

## Using Synonym Sets

You can test your synonym set by searching for one of the synonym keywords you saved in the set. Results matching the keyword and any synonyms are returned in the Search Results widget.

![Try searching for a synonym from your set.](synonym-sets/images/04.png)

In the example above, this blog article about a lunar rover does not contain the word "LRV" but is now returned as a search result match. Note that the synonym is also highlighted.

## Creating New Synonym Language Filters

> **Availability: Liferay DXP 7.3 FP2+ and Liferay DXP 7.2 FP13+**

Out of the box, Synonyms Sets supports synonyms in [English and Spanish only](#requirements-and-limitations). To add support for other languages use the configuration steps below:

- Create a [custom analyzer](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/analysis-custom-analyzer.html) by re-implementing the default [French](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/analysis-lang-analyzer.html#french-analyzer) analyzer with the necessary changes (including an extra [Synonym graph token filter](https://www.elastic.co/guide/en/elasticsearch/reference/7.x/analysis-synonym-graph-tokenfilter.html) in the pipeline)
- Add the custom analyzer definition to the index settings, using the Additional Index Configurations of the connector configuration.
- Apply the custom analyzer on the desired fields by overriding Liferay DXP's default type mappings, using the Override Type Mappings setting of the connector configuration.
- Re-index to apply the new settings.

### Prerequisite: Obtain the Default Mappings

You must obtain the Elasticsearch connector's default JSON mappings before customizing it:

To get the mappings from your Liferay DXP bundle,

1. Go to `[Liferay Home]/osgi/marketplace`.
1. Find the LPKG `Liferay Foundation - Liferay Connector to Elasticsearch 6/7 - Impl.lpkg`.
1. Open with an archive manager, then extract the JAR file that has the name pattern `com.liferay.portal.search.elasticsearch6/7-x.y.z.jar`.
1. Open the extracted JAR file with an archive manager and go to `META-INF/mappings`.
1. The file called `liferay-type-mappings.json` is the resource you need. Extract it to your file system.

To get the mappings from the source code, if you have [Liferay DXP Source Code Access](https://help.liferay.com/hc/en-us/articles/360045389291),

1. Once you have access to the source code repository, follow the steps from the article above to find the tag for your fix pack level.
1. Navigate to `modules/apps/portal-search-elasticsearch(6 or 7)/portal-search-elasticsearch(6 or 7)-impl/src/main/resources/META-INF/mappings`
1. Here you will find the mentioned JSON file (`liferay-type-mappings.json`).

```warning::
   Index mappings and settings can change between versions, and sometimes within a minor version (via a Fix Pack or Service Pack). Customizations to the mappings and settings must be reviewed and adapted as necessary when upgrading or moving to a new patch level. In addition, Liferay's Search team plans to add support for more languages out-of-the-box in future verisons, making customizations unnecessary.
```

### Adding a Language

Now that you have the default mapping file, make the necessary changes to add the necessary settings for French language synonyms:

1. Go to the Elasticsearch connection's System Settings entry---Elasticsearch 6/7.

1. Add an `analysis` block to the Additional Index Configurations field:

   ```json
   {
       "analysis": {
           "analyzer": {
               "custom_liferay_analyzer_fr": {
                   "filter": [
                       "french_elision",
                       "lowercase",
                       "french_stop",
                       "my-synonym-filter-fr",
                       "french_stemmer"
                   ],
                   "tokenizer": "standard"
               }
           },
           "filter": {
               "my-synonym-filter-fr": {
                   "lenient": true, 
                   "synonyms": [],
                   "type": "synonym_graph"
               },
               "french_stop":{
                   "type":"stop",
                   "stopwords":"_french_"
               },
               "french_stemmer":{
                   "type":"stemmer",
                   "language":"light_french"
               },
               "french_elision": {
                   "type": "elision",
                   "articles_case": true,
                   "articles": [
                       "l", "m", "t", "qu", "n", "s",
                       "j", "d", "c", "jusqu", "quoiqu",
                       "lorsqu", "puisqu"
                   ]
               }
           }
       }
   }
   ```

   Adding settings in this configuration augments those available in the out of the box index settings. Find the default JSON settings in the source code's `index-settings.json` file. Here you're creating a new analyzer called `custom_liferay_analyzer_fr` which uses a new filter `my-synonym-filter-fr`. The `synonyms` array is empty for now: Synonym Sets created through the UI will appear here.

1. Using  the Override Type Mappings field, change the analyzer for the `template_fr` dynamic field to use the custom analyzer (`custom_liferay_analyzer_fr`):
   
   ```important::
      This example is clipped for brevity. Override Type Mappings completely overrides and ignores Liferay's default type mappings, so you must provide a complete mappings file, not just the overridden portion.   
   ```

   ```json
    {
    "LiferayDocumentType": {
        "date_detection": false,
        "dynamic_templates": [
            // (...)
            {
                "template_fr": {
                    "mapping": {
                        "analyzer": "custom_liferay_analyzer_fr",
                        "store": true,
                        "term_vector": "with_positions_offsets",
                        "type": "text"
                    },
                    "match": "\\w+_fr\\b|\\w+_fr_[A-Z]{2}\\b",
                    "match_mapping_type": "string",
                    "match_pattern": "regex"
                }
            },
            // (...)
   ```

   The important change here is that the default assigned analyzer (`french`) is replaced with the custom analyzer `custom_liferay_analyzer_fr`.

1. Save the changes to the configuration.

   ```tip::
      If you're using the Sidecar Elasticsearch server, you may see an error in the console. Restart Liferay DXP to resolve the issue.
   ```

1. Now go to System Settings &rarr; Search &rarr; Synonyms.

1. Add the custom filter name (e.g., `custom-synonym-filter-fr`) to the Filter Names setting and save the configuration.

1. Perform a full re-index: in Control Panel &rarr; Search &rarr; Index Actions, click _Reindex all search indexes._

   To verify that the custom mapping has been applied successfully, go to the Field Mappings tab, choose your index (for example `liferay-20101`) and look for `template_fr` in the right panel.

   To verify that the additional index settings including your custom analyzer has also been added, make the following API call to Elasticsearch: `http://<host>:<port>/liferay-[company-id]/_settings` and look for your analyzer name in the response. For example, to see the index settings for the sidecar Elasticsearch server of a Liferay DXP server running at `localhost`, with a `20101`, visit <http://localhost:9201/liferay-20101/_settings>.

To verify the new filter is working, 

1. Go to the Synonyms application: from the Global menu's Applications tab, click _Synonyms_ (under Search Tuning).

1. Create a new Synonym Set: `maison, logement`.

1. Create a Web Content Article with English and French translations. Add _maison_ to the French title.

1. Create another Web Content Article with English and French translations. Add _logement_ to the French title.

1. Switch to the French locale and search for _maison_. Both articles are returned.

