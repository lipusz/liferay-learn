# Custom Filter Examples

The Custom Filter widget is a powerful aid to your search tuning efforts. Without deploying custom code you can exert control over the query sent to the search engine. Some common use cases are presented here to help you understand how to approach the Custom Filter widget:

- Excluding Content from Search Results, based on a field's value
- Boosting Content in Search Results, based on a field's value or presence

See [Filtering Search Results](./filtering-search-results.md) for a detailed explanation of the Custom Filter widget.

## Excluding Certain Content

When used with a `must_not` Occur configuration, the Custom Filter can catch and exclude documents from the Search Results.

### Excluding Certain Documents and Media Content

Sometimes, you may want to exclude certain types of content from appearing in the Search Results. To exclude Documents and Media file entries that are only present in the system to be added to Web Content, you must first distinguish the particular files to exclude, in a way that they can be . You can tag them with something that declares their purpose (`wconly`, perhaps) or you can put them into a dedicated [Documents and Media Folder](./../../../content-authoring-and-management/documents-and-media/uploading-and-managing/creating-folders.md ). To configure a Custom Filter to exclude a Documents and Media Folder, use these settings:

**Filter Query Type:** `Match`

**Filter Field:** `folderId`

**Filter Value:** `41103`

**Occur:** `must_not`

This configuration ensures that search documents containing a `folderId` field with the value `41103` are not returned in the search results.

### Excluding Documents and Media with Certain Extensions

Perhaps you need to exclude GIF files from appearing in the search results. Configure a Custom Filter like this:

**Filter Query Type:** `Match`

**Filter Field:** `extension`

**Filter Value:** `gif`

**Occur:** `must_not`

This configuration leverages the presence of the `extension` field indexed from the Documents and Media [`DLFileEntry` model](https://github.com/liferay/liferay-portal/blob/[$LIFERAY_LEARN_PORTAL_GIT_TAG$]/modules/apps/document-library/document-library-service/src/main/java/com/liferay/document/library/internal/search/spi/model/index/contributor/DLFileEntryModelDocumentContributor.java#L158). It ensures that search documents containing an `extension` field with the value `gif` are not returned in the search results.

## Boosting Fields

Boosting certain documents based on specific fields is a common need, and it's readily accomplished with the Custom Filter widget. The boost value often needs tuning to meet your needs. Use the Search Insights widget, with _Enable Score Explanation_ enabled, to inspect how the documents are being scored and to fine-tune your boost values.

### Boosting Matches to Designated Fields

To boost a document with certain fields that match the searched keywords, configure a Custom Filter like this:

**Filter Query Type:** `Multi Match`

**Filter Field:** `title_en_US, content_en_US`

**Occur:** `Should`

**Boost:** `100`

**Custom Parameter Name:** `q`

This configuration lets you boost matching documents if the English (United States) title and content contains the keywords entered by the user in the Search Bar widget. Entering the Custom Parameter Name with the same value as the Search Bar's Keywords Parameter Name configuration means that the value passed in to the Search Bar is the value that's boosted by the Custom Filter (if it matches with documents in the search index).

Using the Multi Match query allows you to match multiple fields at once. Otherwise you'd need to configure a Custom Filter for each separate field, even if the rest of the configuration values are identical.

### Boosting by a Field's Presence

To boost any content that's tagged, regardless of what the tag value is, configure a Custom Filter like this:

**Filter Query Type:** `Exists`

**Filter Field:** `assetTagNames`

**Occur:** `should`

**Boost:** `100`

If a document matching the query is tagged, it will contain a `assetTagNames` field. The Exists query is used to match based on any value for the field. 

## Related Content

- [Filtering Search Results](./filtering-search-results.md)
- [Result Rankings](../../search-administration-and-tuning/result-rankings.md)
- [Synonym Sets](../../search-administration-and-tuning/synonym-sets.md)
