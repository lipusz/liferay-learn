# Synonym Sets

> **Subscribers**

Synonym Sets are administrator-created groups of words or phrases with the same meaning. If a User searches for a certain keyword or phrase, the synonymous terms in your set are also searched. 

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

Out of the box, Synonyms Sets supports synonyms in [English and Spanish only](#requirements-and-limitations). 

[LET'S DESCRIBE: WHAT DOES A LANGUAGE FILTER DO, AT A HIGH LEVEL?]

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

   [DESCRIBE WHAT THIS DOES AT A HIGH LEVEL]

   ```
   Adding settings in this configuration augments those available in the out of the box [index settings](https://github.com/liferay/liferay-portal/blob/[$LIFERAY_LEARN_PORTAL_GIT_TAG$]/modules/apps/portal-search-elasticsearch7/portal-search-elasticsearch7-impl/src/main/resources/META-INF/index-settings.json).

1. Using  the Override Type Mappings field, override the default `template_fr` to use the custom language filter:
   
   ```important::
      This example is clipped for brevity. Override Type Mappings completely overrides and ignores Liferay's default type mappings, so you must provide a complete mappings file, not just the overridden portion. The fastest way to do this is to copy the `entire mappings file <https://github.com/liferay/liferay-portal/blob/[$LIFERAY_LEARN_PORTAL_GIT_TAG$]/modules/apps/portal-search-elasticsearch7/portal-search-elasticsearch7-impl/src/main/resources/META-INF/mappings/liferay-type-mappings.json>`__ and overwrite the existing ``template_fr`` with the new one.   
   ```
   ```json
   {
   	"LiferayDocumentType": {
   		"date_detection": false,
   		"dynamic_templates": [
   			{
   				"template_ddmFieldArray_ddmFieldValue_Number_sortable": {
   					"mapping": {
   						"scaling_factor": 1000,
   						"store": true,
   						"type": "scaled_float"
   					},
   					"path_match": "ddmFieldArray.ddmFieldValue*Number_sortable"
   				}
   			},
   			{
   				"template_fi": {
   					"mapping": {
   						"analyzer": "finnish",
   						"store": true,
   						"term_vector": "with_positions_offsets",
   						"type": "text"
   					},
   					"match": "\\w+_fi\\b|\\w+_fi_[A-Z]{2}\\b",
   					"match_mapping_type": "string",
   					"match_pattern": "regex"
   				}
   			},
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
   ```

   [DESCRIBE WHAT THIS DOES AT A HIGH LEVEL]


1. Save the changes to the configuration.

   ```tip::
      If you're using the Sidecar Elasticsearch server, you may see an error in the console. Restart Liferay DXP to resolve the issue.
   ```

1. Now go to System Settings &rarr; Search &rarr; Synonyms.

1. Add the filter name (e.g., `custom-synonym-filter-fr`) to the Filter Names setting and save the configuration.

1. Perform a full re-index: in Control Panel &rarr; Search &rarr; Index Actions, click _Reindex all search indexes._

   [SHOULD WE ADD AN API CALL TO FETCH THE INDEX SETTINGS, AND THE MAPPINGS, TO SEE THAT THEY'RE "PERSISTED"?]

To verify the new filter is working, 

1. Go to the Synonyms application: from the Global menu's Applications tab, click _Synonyms_ (under Search Tuning).

1. Create a new Synonym Set: `SP1, FP2`.

1. Create a Web Content Article with English and French translations. Add _SP1_ to the French title.

1. Create another Web Content Article with English and French translations. Add _FP2_ to the French title.

1. Switch to the French locale and search for _FP2_. Both articles are returned.
