# Using the Built-In Elasticsearch

If you obtained Liferay DXP by downloading a Tomcat bundle or pulling a Docker tag, an Elasticsearch node is started with DXP as either an embedded or sidecar server.

If you donwloaded a bundle for a different application server, when you start the server an Elasticsearch distribution is downlaoded on-the-fly and started as a sidecar server.

<!-- nope, not sidecar from what i am hearing -->
Liferay DXP runs an Elasticsearch node in the same JVM so it's easy to test-drive
with minimal configuration. Running both servers in the same process has
drawbacks:

- Elasticsearch must use the same JVM options as Liferay DXP.
- Liferay DXP and Elasticsearch compete for the same system resources. 

```note::
   While it's not a supported production configuration, installing Kibana to monitor the embedded Elasticsearch server is useful during development and testing. Just be aware that you must install the `OSS only Kibana build <https://www.elastic.co/downloads/kibana-oss>`__.
```

You wouldn't run an embedded database like HSQL in production, and you shouldn't run Elasticsearch in embedded mode in production either. Instead, run Elasticsearch in _remote operation mode_, as a standalone server or cluster of server nodes.

