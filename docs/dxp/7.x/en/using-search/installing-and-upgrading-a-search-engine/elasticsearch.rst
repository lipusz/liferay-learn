Elasticsearch
=============

Elasticsearch, a highly scalable, full-text search engine, is the default search engine used by Liferay DXP. Elasticsearch almost certainly meets any search and indexing need you have, but you must not use the `built-in version in your production deployment <./elasticsearch/using-the-sidecar-or-embedded-elasticsearch.md>`__. 

Always consult the `compatibility matrix <https://help.liferay.com/hc/en-us/sections/360002103292-Compatibility-Matrix>`__ to learn which versions of Elasticsearch are supported for your version of Liferay DXP.

.. important::

   Liferay CE/DXP 7.2 and 7.3 support for Elasticsearch 7 is included out of the box, including support for securing the Elasticsearch connection with authentication and encryption. Securing the Elasticsearch 6 connector (available on Liferay CE/DXP 7.2) requires an additional module only available with a `Liferay Enterprise Search subscription <./../liferay_enterprise_search.rst>`__

Installing
~~~~~~~~~~

-  `Getting Started <./elasticsearch/getting-started-with-elasticsearch.md>`__

-  :doc:`/using-search/installing-and-upgrading-a-search-engine/elasticsearch/installing-elasticsearch`
-  :doc:`/using-search/installing-and-upgrading-a-search-engine/elasticsearch/connecting-to-elasticsearch`
-  :doc:`/using-search/installing-and-upgrading-a-search-engine/elasticsearch/exercise-installing-elasticsearch`

- `[Clustering Liferay DXP]` `Add a Search Engine to a Liferay DXP Cluster <../../installation-and-upgrades/setting-up-liferay-dxp/clustering-for-high-availability/example-creating-a-simple-dxp-cluster.md#prepare-a-search-engine>`__

-  :doc:`/using-search/installing-and-upgrading-a-search-engine/elasticsearch/troubleshooting-elasticsearch-installation`

Install an Older Elasticsearch Version
______________________________________

- Elasticsearch 6 (Coming Soon!)

Securing
~~~~~~~~

Coming Soon!

Upgrading
~~~~~~~~~

Coming Soon!

Configuring
~~~~~~~~~~~

Coming Soon!

Liferay Enterprise Search
~~~~~~~~~~~~~~~~~~~~~~~~~

Coming Soon!

.. toctree::
   :maxdepth: 2

   elasticsearch/getting-started-with-elasticsearch.md
   elasticsearch/installing-elasticsearch.md
   elasticsearch/connecting-to-elasticsearch.md
   elasticsearch/troubleshooting-elasticsearch-installation.md
   elasticsearch/exercise-installing-elasticsearch.md
   elasticsearch/using-the-sidecar-or-embedded-elasticsearch.md
