Elasticsearch
=============

Elasticsearch, a highly scalable, full-text search engine, is installed by default, as an embedded server (7.3 CE GA3 and earlier) or as a sidecar server (7.3 CE GA4 and later). Elasticsearch is well-supported and almost certainly meets any search and indexing need you have, but you must not use the `embedded version in your production deployment <./installing-a-search-engine.md>`__. 

.. important::
   Always consult the `compatibility matrix <https://www.liferay.com/documents/10182/246659966/Liferay+DXP+7.2+Compatibility+Matrix.pdf/ed234765-db47-c4ad-7c82-2acb4c73b0f9>`__ to learn which versions of Elasticsearch are supported for your version of Liferay DXP.

Operation Mode
~~~~~~~~~~~~~~

When you start Liferay DXP, an Elasticsearch node is started simultaneously as either an embedded or sidecar server.

Liferay DXP runs an Elasticsearch node in the same JVM so it's easy to test-drive
with minimal configuration. Running both servers in the same process has
drawbacks:

- Elasticsearch must use the same JVM options as Liferay DXP.
- Liferay DXP and Elasticsearch compete for the same system resources. 

.. note::
   While it's not a supported production configuration, installing Kibana to monitor the embedded Elasticsearch server is useful during development and testing. Just be aware that you must install the [OSS only Kibana build](https://www.elastic.co/downloads/kibana-oss).

You wouldn't run an embedded database like HSQL in production, and you shouldn't run Elasticsearch in embedded mode in production either. Instead, run Elasticsearch in `remote operation mode`, as a standalone server or cluster of server nodes.

Installing
~~~~~~~~~~

-  `Install the Latest Supported Elasticsearch Version <./elasticsearch/installing-elasticsearch.md>`__

- `[Clustering Liferay DXP]` `Add a Search Engine to a Liferay DXP Cluster <../../installation-and-upgrades/setting-up-liferay-dxp/clustering-for-high-availability/example-creating-a-simple-dxp-cluster.md#prepare-a-search-engine>`__

-  :doc:`/using-search/installing-and-upgrading-a-search-engine/elasticsearch/exercise-upgrading-the-example-liferay-dxp-cluster`

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

   elasticsearch/installing-elasticsearch.md
   elasticsearch/troubleshooting-elasticsearch-installation.md
   elasticsearch/exercise-upgrading-the-example-liferay-dxp-cluster.md
