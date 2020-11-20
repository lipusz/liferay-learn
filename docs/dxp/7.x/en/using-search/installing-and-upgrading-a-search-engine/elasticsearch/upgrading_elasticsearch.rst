Upgrading Elasticsearch
=======================

.. toctree::
   :maxdepth: 2
   :glob:

   upgrading-elasticsearch/*


Liferay 7.3 brings `new improvements <../getting-started/whats-new-in-search-for-73.md>`__ for search,including built-in support for Elasticsearch 7. The `compatibility matrix <https://help.liferay.com/hc/en-us/sections/360002103292-Compatibility-Matrix>`__ provides the latest support details.

.. important::
   Solr integration is now deprecated as of Liferay 7.3. Elasticsearch integration replaces it. Migrating to Elasticsearch requires `setting up Elasticsearch `<./elasticsearch/getting-started-with-elasticsearch.md>`_ and `connecting Liferay <./elasticsearch/connecting-to-elasticsearch.md`_ to it.

.. important::
   Elasticsearch 6.x is not supported on Liferay 7.3.

Read `Upgrading Search for Liferay 7.3 <./upgrading-search-for-liferay-73.md>`__ first. It summarizes upgrading from search stacks on previous Liferay versions to the latest supported search stack. Then `back up your search indexes <./backing-up-elasticsearch.md>`__ before `upgrading Elasticsearch <./upgrading-to-elasticsearch-7.md>`__.

-  :doc:`/using-search/installing-and-upgrading-a-search-engine/elasticsearch/upgrading-elasticsearch/upgrading-search-for-liferay-73`
-  :doc:`/using-search/installing-and-upgrading-a-search-engine/elasticsearch/upgrading-elasticsearch/upgrading-to-elasticsearch-7`
-  :doc:`/using-search/installing-and-upgrading-a-search-engine/elasticsearch/upgrading-elasticsearch/backing-up-elasticsearch`
