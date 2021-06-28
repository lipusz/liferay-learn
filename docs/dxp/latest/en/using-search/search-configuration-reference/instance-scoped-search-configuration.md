# Instance Scoped Search Configuration

Search does not have any Instance scoped entries in the [Instance Settings](../../system-administration/configuring-liferay/understanding-configuration-scope.md#system-settings-and-instance-settings) panel in Liferay 7.3 and below. However, there are entries for configuring the instance-wide default [widget template](../../site-building/displaying-content/additional-content-display-options/styling-widgets-with-widget-templates.md) for many search widgets. Just enter the Site ID where the template is defined (often this will be the Global site's ID, found in its Site Settings panel) and the Widget Template ID, found in the Site Menu &rarr; Design &rarr; Widget Templates.

The only instance scoped configurations for search are the entries in the Global Menu &rarr; Applications &rarr; Search Tuning:

**Result Rankings:** Customize search results manually by hiding, pinning, and adding results for specific aliases. See [Result Rankings](../search-administration-and-tuning/result-rankings.md) for more information.

**Synonyms:** Create synonym sets so that synonymous search terms are matched and scored like an exact match to the search term. See [Synonym Sets](../search-administration-and-tuning/synonym-sets.md) for more information.
