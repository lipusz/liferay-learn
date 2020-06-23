# Using the Built-In Elasticsearch

If you obtained Liferay DXP by downloading a Tomcat bundle or pulling a Docker tag, an Elasticsearch node is started with DXP as either an embedded or sidecar server. To confirm,

- If running Liferay DXP 7.2, access the embedded server at <http://localhost:9200>
- If running Liferay DXP 7.3 GA4+, access the sidecar server at <http://localhost:9201>

You'll see information like this:

<!-- THIS IS MY REMOTE 7.7-REPLACE WITH A SIDECAR SERVER'S OUTPUT-->

```json
{
  "name" : "400ec372f7ca",
  "cluster_name" : "LiferayElasticsearchCluster",
  "cluster_uuid" : "SpeO6mpHQpiMyKGab2X8Zg",
  "version" : {
    "number" : "7.7.1",
    "build_flavor" : "default",
    "build_type" : "docker",
    "build_hash" : "ad56dce891c901a492bb1ee393f12dfff473a423",
    "build_date" : "2020-05-28T16:30:01.040088Z",
    "build_snapshot" : false,
    "lucene_version" : "8.5.1",
    "minimum_wire_compatibility_version" : "6.8.0",
    "minimum_index_compatibility_version" : "6.0.0-beta1"
  },
  "tagline" : "You Know, for Search"
}
```

<!-- nope, not sidecar from what i am hearing -->
Liferay DXP runs an Elasticsearch node in the same JVM so it's easy to test-drive with minimal configuration. Running both servers in the same process has drawbacks:

- Elasticsearch must use the same JVM options as Liferay DXP.
- Liferay DXP and Elasticsearch compete for the same system resources. 

```note::
   While it's not a supported production configuration, installing Kibana to monitor the embedded Elasticsearch server is useful during development and testing. Just be aware that you must install the `OSS only Kibana build <https://www.elastic.co/downloads/kibana-oss>`__.
```

You wouldn't run an embedded database like HSQL in production, and you shouldn't run Elasticsearch in embedded mode in production either. Instead, run Elasticsearch in [_remote operation mode_](./installing-elasticsearch.md), as a standalone server or cluster of server nodes.

## Default Elasticsearch and App Server Differences

While an Elasticsearch sidecar server is pre-installed for Liferay DXP 7.3 GA4+ Tomcat bundles and Docker images, there are some key differences if you're installing a different bundle or installing the Lifery DXP WAR onto WebSphere or Weblogic application servers.

| Liferay DXP Flavor       | Default Elasticsearch | Pre-Installed | Requires Manual Intervention |
| ------------------------ | ------------------- | ------------- | ---------------------------- |
| Tomcat bundle: 7.3 GA4+  | Sidecar             | &#10004;      | &#10008;                     |
| WebSphere: 7.3 GA4+      | Sidecar             | &#10008;      | &#10004;                     |
| Weblogic: 7.3 GA4+       | Sidecar             | &#10008;      | &#10004;                     |
| Tomcat bundle: 7.2       | Embedded            | &#10004;      | &#10008;                     |



  differif you downloaded a bundle for an application server besides Tomcat, when you start the server an Elasticsearch distribution is downloaded on-the-fly and started as a sidecar server.
<!--Talk briefly about Websphere and Weblogic and link to those docs -->

```important::
   The built-in Elasticsearch server is useful for development and testing purposes and must not be used in production. See `Installing Elasticsearch <./installing-elasticsearch.md>`__ to learn about installing a REMOTE mode search engine.
```

## Embedded versus Sidecar


Common activities the default Elasticsearch server can be used for include

- Testing your custom [search and indexing code](../../developer-guide/search-and-indexing.md)
- Developing search queries by running queries directly on Elasticsearch through Kibana
- Testing the [search tuning](../../search_administration_and_tuning.rst) functionality
- Exploring and configuring the [search widgets](../../search_pages_and_widgets.rst)
