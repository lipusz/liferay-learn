# Configuring CCR: Settings Reference

To help with the CCR configuration process, the example configurations from the step-by-step instructions in this guide are collected here. These configuration are made as generic as possible, but paths, ports, etc. will need to be adjusted to match your environment. In addition, you must perform certain steps manually to complete the installation, so these cannot replace the step-by-step instructions.

The configurations below assume you enable encrypted communications (TLS/SSL) and user authentication through X-Pack Security in your installation.

```tip::
   `Configuration values provided by .config files <../../../system-administration/system-settings/using-configuration-files.md>`__ are propagated throughout a DXP cluster as soon as the file is deployed to a single node. However, it's a best practice to provide identical configurations for each cluster node. 

   You might notice that the instructions in this guide appear to violate the above statement. The CCR specific configuration files are not provided to the remote/leader Liferay DXP nodes. Because the CCR module is not deployed to these nodes, the configurations are inert and can be disregarded.
```

## Remote DXP Cluster Node Configurations

These configuration files are deployed to `[Remote Liferay Home]/osgi/configs`.

### Remote DXP Cluster Node Configurations for Elasticsearch 7

The Remote DXP cluster needs three configuration files.

File name: `com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConfiguration.config`

File contents:

```properties
productionModeEnabled="true"
networkHostAddresses=["http://localhost:9200"]
logExceptionsOnly="false"
authenticationEnabled="true"
username="elastic"
password="liferay"
httpSSLEnabled="true"
networkHostAddresses=["https://localhost:9200"]
truststorePassword="liferay"
truststorePath="/PATH/TO/elastic-nodes.p12"
truststoreType="pkcs12"
```

```tip::
   The X-Pack security configuration file is not required on Liferay DXP 7.3. Security is configured in the ``ElasticsearchConfiguration.config`` and in each connection's ``.config`` file. The values should be identical on the remote/leader nodes and the local/follower nodes.
```

File name: `com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConnectionConfiguration-remote.config`

File contents:

```properties
active="true"
connectionId="remote"
networkHostAddresses=["http://localhost:9200"]
authenticationEnabled="true"
username="elastic"
password="liferay"
httpSSLEnabled="true"
truststorePassword="liferay"
truststorePath="/PATH/TO/elastic-nodes.p12"
truststoreType="pkcs12"
```

File name: `com.liferay.portal.search.elasticsearch7.configuration.ElasticsearchConnectionConfiguration-ccr.config`

File contents:

```properties
active="true"
connectionId="ccr"
networkHostAddresses=["http://localhost:9202"]
authenticationEnabled="true"
username="elastic"
password="liferay"
httpSSLEnabled="true"
truststorePassword="liferay"
truststorePath="/PATH/TO/elastic-nodes.p12"
truststoreType="pkcs12"
```

## Local DXP Cluster Node Configurations

These configuration files are deployed to `[Local Liferay Home]/osgi/configs`.

### Local DXP Cluster Node Configurations for Elasticsearch 7

Provide identical Elasticsearch 7 configurations (including the connection configuration files) to the local DXP nodes that you provided to the remote DXP nodes.

### Local DXP Cluster Node CCR Module Configurations

File name: ` com.liferay.portal.search.elasticsearch.cross.cluster.replication.internal.configuration.CrossClusterReplicationConfiguration.config`

File contents:

```properties
ccrEnabled="true"
ccrLocalClusterConnectionConfigurations=[ \
  "localhost:8080,ccr", \
  "localhost:9080,ccr", \
  ]
```

## Leader Elasticsearch Cluster Node Configurations

Location: `ES_LEADER_HOME/config`

File name: `elasticsearch.yml`

File contents:
```yaml
cluster.name: LiferayElasticsearchCluster_LEADER
node.name: es-leader-node-1

http.port: 9200

xpack.security.enabled: true

## TLS/SSL settings for Transport layer
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certs/elastic-nodes.p12
xpack.security.transport.ssl.keystore.password: liferay
xpack.security.transport.ssl.truststore.path: certs/elastic-nodes.p12
xpack.security.transport.ssl.truststore.password: liferay

# TLS/SSL settings for HTTP layer
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.verification_mode: certificate
xpack.security.http.ssl.keystore.path: certs/elastic-nodes.p12
xpack.security.http.ssl.keystore.password: liferay
xpack.security.http.ssl.truststore.path: certs/elastic-nodes.p12
xpack.security.http.ssl.truststore.password: liferay

# For Kibana
xpack.monitoring.collection.enabled: true
```

## Follower Elasticsearch Cluster Node Configurations

Location: `ES_FOLLOWER_HOME/config`

File name: `elasticsearch.yml`

File contents:
```yaml
cluster.name: LiferayElasticsearchCluster_FOLLOWER
node.name: es-follower-node-1

http.port: 9202

xpack.security.enabled: true

## TLS/SSL settings for Transport layer
xpack.security.transport.ssl.enabled: true
xpack.security.transport.ssl.verification_mode: certificate
xpack.security.transport.ssl.keystore.path: certs/elastic-nodes.p12
xpack.security.transport.ssl.keystore.password: liferay
xpack.security.transport.ssl.truststore.path: certs/elastic-nodes.p12
xpack.security.transport.ssl.truststore.password: liferay

# TLS/SSL settings for HTTP layer
xpack.security.http.ssl.enabled: true
xpack.security.http.ssl.verification_mode: certificate
xpack.security.http.ssl.keystore.path: certs/elastic-nodes.p12
xpack.security.http.ssl.keystore.password: liferay
xpack.security.http.ssl.truststore.path: certs/elastic-nodes.p12
xpack.security.http.ssl.truststore.password: liferay

# For Kibana
xpack.monitoring.collection.enabled: true
```
